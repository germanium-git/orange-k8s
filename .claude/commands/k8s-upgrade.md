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

## Instructions

Follow these steps in order. Run each SSH command via the Bash tool. Stop and report if any step fails.

### Step 0 — Determine Target Version

If the user has not provided a target version in `$ARGUMENTS`, ask:
- Target **minor** version (e.g. `1.31`)
- Target **patch** version (e.g. `1.31.5`)

Derive the apt package suffix: `<patch>-1.1` (e.g. `1.31.5-1.1`).

Variables used throughout:
- `MINOR` = minor version (e.g. `1.31`)
- `VERSION` = full patch version (e.g. `1.31.5`)
- `PKG` = apt package version string (e.g. `1.31.5-1.1`)

### Step 1 — Pre-flight Health Check

```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "kubectl get nodes && echo '---' && kubectl get pods -A | grep -Ev '(Running|Completed)'"
```

- All nodes must show `Ready`.
- Any non-daemonset pod not in Running/Completed state must be investigated before proceeding.
- Abort and report if the cluster is unhealthy.

### Step 2 — Confirm Version Availability

```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "apt-cache madison kubeadm 2>/dev/null | grep \" ${VERSION}-\" | head -3"
```

Confirm that the target `${PKG}` package is listed. If not, show available versions and ask the user to choose.

---

### Step 3 — Upgrade Control Plane (orange1)

Run all commands in this step via SSH to orange1 (`petr@172.31.1.50`). Use `-t` flag when sudo interactive input might be needed.

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
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo apt-get update -q"
```

**3d. Upgrade kubeadm:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo apt-mark unhold kubeadm && sudo apt-get install -y kubeadm=${PKG} && sudo apt-mark hold kubeadm"
```

**3e. Review upgrade plan:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo kubeadm upgrade plan"
```

Show the output to the user and ask for confirmation before applying.

**3f. Apply upgrade:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo kubeadm upgrade apply v${VERSION} --yes"
```

**3g. Drain control plane node:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "kubectl drain orange1 --ignore-daemonsets --delete-emptydir-data"
```

**3h. Upgrade kubelet and kubectl:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo apt-mark unhold kubelet kubectl && sudo apt-get install -y kubelet=${PKG} kubectl=${PKG} && sudo apt-mark hold kubelet kubectl"
```

**3i. Reload and restart kubelet:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "sudo systemctl daemon-reload && sudo systemctl restart kubelet"
```

**3j. Uncordon and verify:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "kubectl uncordon orange1 && kubectl get nodes"
```

Confirm orange1 shows `Ready` and the correct version before continuing.

---

### Step 4 — Upgrade Worker Nodes (one at a time)

Repeat the following sub-steps for **orange2** (172.31.1.51) then **orange3** (172.31.1.52).

Replace `WORKER_HOST` and `WORKER_IP` with the current node's values.

**4a. Update apt repo + GPG key on worker:**
```bash
ssh -i ~/.ssh/orangePiMac petr@${WORKER_IP} "echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v${MINOR}/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list && curl -fsSL https://pkgs.k8s.io/core:/stable:/v${MINOR}/deb/Release.key | sudo gpg --yes --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg && sudo apt-get update -q"
```

**4b. Upgrade kubeadm on worker:**
```bash
ssh -i ~/.ssh/orangePiMac petr@${WORKER_IP} "sudo apt-mark unhold kubeadm && sudo apt-get install -y kubeadm=${PKG} && sudo apt-mark hold kubeadm"
```

**4c. Drain worker from control plane:**
```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "kubectl drain ${WORKER_HOST} --ignore-daemonsets --delete-emptydir-data"
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
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "kubectl uncordon ${WORKER_HOST} && kubectl get nodes"
```

Wait until `${WORKER_HOST}` shows `Ready` before starting the next worker node.

---

### Step 5 — Post-upgrade Verification

```bash
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50 "kubectl get nodes && echo '---' && kubectl get pods -A"
```

- All three nodes must show `Ready` with the new version.
- Report any pods not in Running/Completed state.
- Summarize the upgrade result to the user.
