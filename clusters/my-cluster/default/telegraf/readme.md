# Telegraf

## Installation

### SNMP MIBs

Upload the SNMP MIBs stored in the mibs folder to /var/tmp/telegraf on each K8s node.

```
scp -i ~/.ssh/privkey -r mibs username@host:/var/tmp/telegraf
```

```
❯ ls /var/tmp/telegraf -al
total 12
drwxrwxr-x 3 petr petr 4096 Aug 31 14:36 .
drwxrwxrwt 8 root root 4096 Sep  1 00:29 ..
drwxr-xr-x 4 petr petr 4096 Aug 31 14:36 mibs
```

### Secrets

Create a yaml file telegraf-secrets.yaml and fill in all respective secrets, at the moment only one is used INFLUXDB_TOKEN.

```
apiVersion: v1
kind: Secret
metadata:
  name: telegraf-secrets
  namespace: telegraf
type: Opaque
stringData:
  INFLUXDB_TOKEN: xxxxxxxxxxxxxxxxxxxxxx

```

```
cat telegraf-secrets.yaml| kubeseal --format=yaml > telegraf-sealedsecret.yaml
```

### Other links

https://octoperf.com/blog/2019/09/19/kraken-kubernetes-influxdb-grafana-telegraf/#how-to-deploy-telegraf


## Don't forget

Allow SNMP query for the each IP address of the K8s nodes on the target router.

```
set snmp community public clients <IP address>
```