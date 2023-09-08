# Metric server

## Installation

One option is to install metric API from kubernetes git

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

or download the respective components.yaml file and split it to multiple parts have flux deploy the resources.

## Issues

To address the problem indicated in the logs of the metric-server deployment referring to nodes using IP addresses instead of names

```
...error while getting metrics ...x509: cannot validate certificate for
```

add the following parameter to the deployment as recommended at https://github.com/kubernetes-sigs/metrics-server/issues/196

```
containers:
      - args:
        - --kubelet-insecure-tls
```