# MotionEye

Use the K8s manaifests published at https://github.com/tmoss343/kubernetes-motioneye

## Customization

### motioneye-deployment.yaml
- namespace: motioneye
- resources requests/limits
- image for ARM ccrisan/motioneye:master-armhf

### motioneye-pv-claims.yaml
- storageClassName: nfs-client

### motioneye-service.yaml
- namespace: motioneye
- type: ClusterIP

### Custom manifests
- motioneye-namespace.yaml
- motioneye-ingressroute.yaml

### Unused manifests
- motioneye-ingress.yaml
- motioneye-pv-volume.yaml

## Installation

Have FluxCD install all manifests stored in the motioneye folder.
The IngressRoute *motioneye-ingressroute.yaml* to access MotionEye UI assumes Traefik already running on the cluster as the Ingress controller.

## Access WebUI

The UI is exposed through the IngressRoute `motioneye-ingressroute.yaml`
username admin, no passsord

```
http://motioneye.germanium.home:8000/
```

## Backup Config

Download the file `motioneye-config.tar.gz` from UI and save it to the Google Drive

## Restore

If the restore does not work through UI follow the instructions below to make it happen.

* Untar/unzip the backup file `motioneye-config.tar.gz` (motioneye-config-k8s-orange.tar.gz from the Google drive)
    * Path Google\ Drive/My\ Drive/PrivateDocuments/Dokumentace/MotionEye/motioneye-config-k8s-orange.tar.gz
* Copy all the files to the container

```
kubectl cp ~/Downloads/motioneye-config-k8s-orange/. motioneye/motioneye-6c67667f49-h5lt9:/etc/motioneye/ -c motioneye
```

* Copy the files to the mounted PVC. Ignore the error messages about ownership.

```
root@motioneye-6c67667f49-t2b5p:/# ls -al /etc/motioneye/
total 840
drwxrwxrwx 2 nobody nogroup   4096 Oct 13 10:16 .
drwxr-xr-x 1 root   root      4096 Oct 13 10:20 ..
-rw-r--r-- 1 nobody nogroup   2186 Oct 13 10:12 camera-1.conf
-rw-r--r-- 1 nobody nogroup   2222 Oct 13 10:12 camera-2.conf
-rw-r--r-- 1 nobody nogroup 225295 Oct 13 10:12 mask_1.pgm
-rw-r--r-- 1 nobody nogroup 589840 Oct 13 10:12 mask_2.pgm
-rw-r--r-- 1 nobody nogroup    247 Oct 13 10:12 motion.conf
-rw-r--r-- 1 nobody nogroup   2865 Oct 13 10:12 motioneye.conf
-rw-r--r-- 1 nobody nogroup      6 Oct 13 10:21 tasks.pickle
```

Kill the pod and have K8s create another one.
