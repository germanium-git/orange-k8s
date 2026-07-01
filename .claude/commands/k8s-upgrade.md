# K8s Cluster Upgrade

Upgrade the Orange Pi 5 Kubernetes cluster (orange1/2/3) to a new version using SSH.

## Cluster

| Node | Role | IP | Hostname |
|------|------|----|----------|
| orange1 | control plane | 172.31.1.50 | orange1 |
| orange2 | worker | 172.31.1.51 | orange2 |
| orange3 | worker | 172.31.1.52 | orange3 |

SSH: `ssh -i ~/.ssh/orangePiMac petr@<IP>`  
OS: Armbian (Debian-based), ARM64

**kubectl note:** The `petr` user kubeconfig may have expired credentials. Use `sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl` for all kubectl commands on orange1 throughout this procedure.

## Instructions

Follow these steps in order. Run each SSH command via the Bash tool. Stop and report if any step fails.

### Step 0 — Determine Target Version

First check the current cluster version (`kubectl get nodes`). **kubeadm does not support skipping minor versions** — it can only upgrade one minor version at a time (e.g. 1.32 → 1.33, never 1.32 → 1.34 directly). If the user asks for a target more than one minor ahead of the current version, tell them this isn't supported and confirm upgrading to current+1 as this run's target; the remaining minors need separate, later runs.

If the user has not provided a target version in `$ARGUMENTS`, ask:
- Target **minor** version (e.g. `1.31`) — default to current+1
- Target **patch** version (e.g. `1.31.5`) — if unsure, check `apt-cache madison kubeadm` after switching the repo (step 3a–3c) and default to the latest patch listed

Derive the apt package suffix: `<patch>-1.1` (e.g. `1.31.5-1.1`).

Variables used throughout:
- `MINOR` = minor version (e.g. `1.31`)
- `VERSION` = full patch version (e.g. `1.31.5`)
- `PKG` = apt package version string (e.g. `1.31.5-1.1`)

### Step 1 — Pre-flight Health Check

```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl get nodes && echo '---' && sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl get pods -A | grep -Ev '(Running|Completed)'"
```

- All nodes must show `Ready`.
- Any non-daemonset pod not in Running/Completed state must be investigated before proceeding.
- Abort and report if the cluster is unhealthy.

### Step 2 — Confirm Version Availability

After updating the repo in step 3a–3c the new packages become visible. Run a pre-check now to see what the current repo offers:

```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "apt-cache policy kubeadm 2>/dev/null | head -20"
```

If the target `${PKG}` is not listed yet (expected — repo still points at the old minor), note that and proceed to step 3a which will switch the repo. Re-check availability after step 3c.

---

### Step 3 — Upgrade Control Plane (orange1)

Run all commands in this step via SSH to orange1 (`petr@172.31.1.50`).

**3a. Update apt repository to new minor version:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v${MINOR}/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list"
```

**3b. Update GPG key (non-interactive):**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "curl -fsSL https://pkgs.k8s.io/core:/stable:/v${MINOR}/deb/Release.key | sudo gpg --yes --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg"
```

**3c. Update package index:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo apt-get update -q 2>&1 | tail -3"
```

Wait for the command to complete (do not run in background — a concurrent apt lock will cause the next step to fail).

Verify the target version is now available:
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "apt-cache policy kubeadm 2>/dev/null | grep -A5 'Version table'"
```

If `${PKG}` is not listed, show the available versions and ask the user to choose before continuing.

**3d. Upgrade kubeadm:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo apt-mark unhold kubeadm && sudo apt-get install -y kubeadm=${PKG} && sudo apt-mark hold kubeadm"
```

**3e. Review upgrade plan:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo kubeadm upgrade plan"
```

Show the output to the user. **Important:** kubeadm always targets the latest patch in the minor series. If the plan shows a different version than `${VERSION}` (e.g. it recommends `v1.31.14` but you requested `v1.31.9`), ask the user:
- Apply the recommended latest patch instead (requires upgrading kubeadm to that version first), or
- Keep the originally requested version (the installed kubeadm can still apply it with `kubeadm upgrade apply v${VERSION}`)

Update `VERSION` and `PKG` if the user chooses the latest.

**3f. Apply upgrade:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo kubeadm upgrade apply v${VERSION} --yes"
```

**3g. Drain control plane node:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl drain orange1 --ignore-daemonsets --delete-emptydir-data"
```

If drain fails with "cannot delete Pods that declare no controller", those standalone pods must be migrated to a Deployment or deleted manually before proceeding. If drain blocks indefinitely due to a PodDisruptionBudget (repeated "Cannot evict pod as it would violate the pod's disruption budget" messages), cancel and retry with `--disable-eviction` — this bypasses PDB checks and directly deletes pods, which are then rescheduled by their controllers.

**3h. Upgrade kubelet and kubectl:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo apt-mark unhold kubelet kubectl && sudo apt-get install -y kubelet=${PKG} kubectl=${PKG} && sudo apt-mark hold kubelet kubectl"
```

**3i. Reload and restart kubelet:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo systemctl daemon-reload && sudo systemctl restart kubelet"
```

**3j. Wait for API server, then uncordon and verify:**

The kubelet restart briefly takes the API server down. Poll until it responds:
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "for i in {1..12}; do sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl get nodes 2>/dev/null && break || echo 'waiting...'; sleep 5; done"
```

Once orange1 appears `Ready,SchedulingDisabled`, uncordon:
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl uncordon orange1 && sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl get nodes"
```

Confirm orange1 shows `Ready` and the correct version before continuing.

---

### Step 4 — Upgrade Worker Nodes (one at a time)

Repeat the following sub-steps for **orange2** (172.31.1.51) then **orange3** (172.31.1.52).

Replace `WORKER_HOST` and `WORKER_IP` with the current node's values.

**4a. Update apt repo + GPG key on worker:**
```bash
ssh -i ~/.ssh/orangePiMac petr@${WORKER_IP} "echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v${MINOR}/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list && curl -fsSL https://pkgs.k8s.io/core:/stable:/v${MINOR}/deb/Release.key | sudo gpg --yes --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg && sudo apt-get update -q 2>&1 | tail -3"
```

**4b. Upgrade kubeadm on worker:**
```bash
ssh -i ~/.ssh/orangePiMac petr@${WORKER_IP} "sudo apt-mark unhold kubeadm && sudo apt-get install -y kubeadm=${PKG} && sudo apt-mark hold kubeadm"
```

**4c. Drain worker from control plane:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl drain ${WORKER_HOST} --ignore-daemonsets --delete-emptydir-data"
```

If drain fails with "cannot delete Pods that declare no controller", migrate or delete those standalone pods first.

If drain blocks indefinitely on PodDisruptionBudget violations (single-replica apps with `minAvailable: 1` have 0 allowed disruptions), cancel and retry with `--disable-eviction`:
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl drain ${WORKER_HOST} --ignore-daemonsets --delete-emptydir-data --disable-eviction"
```

**4d. Upgrade node config on worker:**
```bash
ssh -i ~/.ssh/orangePiMac petr@${WORKER_IP} "sudo kubeadm upgrade node"
```

**4e. Upgrade kubelet and kubectl on worker:**
```bash
ssh -i ~/.ssh/orangePiMac petr@${WORKER_IP} "sudo apt-mark unhold kubelet kubectl && sudo apt-get install -y kubelet=${PKG} kubectl=${PKG} && sudo apt-mark hold kubelet kubectl"
```

**4f. Reload and restart kubelet on worker:**
```bash
ssh -i ~/.ssh/orangePiMac petr@${WORKER_IP} "sudo systemctl daemon-reload && sudo systemctl restart kubelet"
```

**4g. Uncordon from control plane and verify Ready:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl uncordon ${WORKER_HOST} && sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl get nodes"
```

Wait until `${WORKER_HOST}` shows `Ready` before starting the next worker node.

---

### Step 5 — Post-upgrade Verification

```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl get nodes && echo '---' && sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl get pods -A"
```

- All three nodes must show `Ready` with the new version.
- Report any pods not in Running/Completed state. Pods rescheduled during a drain (e.g. still `Init`) can take a minute or so to settle — re-run the check once before treating it as a real problem.
- Summarize the upgrade result to the user.
