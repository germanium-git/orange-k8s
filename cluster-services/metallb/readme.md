# MetalLB
## Installation

Follow the official instructions at https://metallb.universe.tf/installation/

### Enable strict ARP mode.

```
kubectl edit configmap -n kube-system kube-proxy
```

```
    ipvs:
      excludeCIDRs: null
      minSyncPeriod: 0s
      scheduler: ""
      strictARP: true
```

### Install with Manifest

At the time of writing the installation with Kustomize appears to supports only the BGP mode.

Follow the [instructions](https://metallb.universe.tf/installation/#installation-by-manifest) to install MetalLB.
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml

````
## Configuration

To configure the address pool and L2 mode the follwoing two manifest files have to be created and get applied through kustomization in FluxCD.

### Address Pool
Modify the file *address-pool.yaml* to create another pool.

### L2 mode
Enable L2 mode in the *l2advertisment.yaml* file for each pool. Alternatively remove the section spec/ipAddressPools to enable L2 mode automatically for all of them.