# NFS Provisioner

## Installation

### Helm

Follow the instructions at https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=172.31.1.5 \
    --set nfs.path=/volume1/k8s
```

### Kustomization & FluxCD

To avoid using the remote resources used in the original instructions published at GitHub repo https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/tree/master#with-kustomize copy the files from https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/tree/master/deploy to the nfs-provisioner folder and update the part referred to as patch_nfs_details.yaml in the local deployment.yaml manifest with respective values.

```
      containers:
        - name: nfs-client-provisioner
          env:
            - name: NFS_SERVER
              value: <YOUR_NFS_SERVER_IP>
            - name: NFS_PATH
              value: <YOUR_NFS_SERVER_SHARE>
      volumes:
        - name: nfs-client-root
          nfs:
            server: <YOUR_NFS_SERVER_IP>
            path: <YOUR_NFS_SERVER_SHARE>
```

Having added resource limits the whole section shoudl look as follows:

```
      containers:
        - name: nfs-client-provisioner
          image: registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
          resources:
            requests:
              memory: 32Mi
              cpu: 24m
            limits:
              memory: 64Mi
              cpu: 48m
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: 172.31.1.5
            - name: NFS_PATH
              value: /volume1/k8s
      volumes:
        - name: nfs-client-root
          nfs:
            server: 172.31.1.5
            path: /volume1/k8s
```

## Access from Mac

Mount the NFS share

```
sudo mount -o rw -t nfs 172.31.1.5:/volume1/k8s /private/nfs
```

Check the PVCs from the CLI

```
❯ ls -al /private/nfs
total 40
drwxrwxrwx  6 root  wheel  4096 Sep  1 14:50 .
drwxr-xr-x  7 root  wheel   224 Aug 25 14:56 ..
drwxrwxrwx  3 root  wheel  4096 Aug 25 14:53 @eaDir
drwxrwxrwx  2 root  wheel  4096 Aug 25 15:40 default-test-app-pvc-49643c84-0548-43f1-ae5b-8466a0d299c6
drwxrwxrwx  2 root  wheel  4096 Sep  1 16:02 motioneye-motioneye-etc-pv-claim-pvc-b8d89d58-8401-49ba-8188-bf1ae46f76c7
drwxrwxrwx  2 root  wheel  4096 Sep  1 14:50 motioneye-motioneye-lib-pv-claim-pvc-eff3e694-87fd-47bf-9dfe-dc4ddde537cc
```