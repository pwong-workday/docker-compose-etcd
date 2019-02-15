# docker-compose-etcd

## Prerequisites

This repo uses a submodule. Consequently, you must run:
```bash
git submodule update --init --recursive
```

## How to use

[Docker Compose](https://docs.docker.com/compose/) configuration and scripts for experimenting with [etcd](https://github.com/coreos/etcd) and [etcd-browser](https://github.com/henszey/etcd-browser).

To run etcd and etcd-browser run:
```bash
docker-compose up -d
```

To validate that etcd started correctly run:
```bash
etcd=$(bin/service_address.sh etcd0 2379)
curl $etcd/v2/keys
curl $etcd/v2/keys/foo -XPUT -d value=bar

etcd_all="{$(bin/service_address.sh etcd0 2379),$(bin/service_address.sh etcd1 2379),$(bin/service_address.sh etcd2 2379)}"
curl $etcd_all/v2/keys

curl $etcd_all/v2/stats/leader
```

If you are using firefox you can view your new index in etcd-browser by running ```bin/firefox_etcd.sh```.

This setup and the example scripts should be compatible with any of the following scenarios:
- Local Docker instance
- Remote [Docker Machine](https://docs.docker.com/machine/) instance
- Remote [Docker Swarm](https://docs.docker.com/swarm/) cluster

> Written with [StackEdit](https://stackedit.io/).

## Testing k8s-apiserver audit policy

```bash
# put your audit policy into ./myaudit/audit-policy.yaml

docker pull quay.io/coreos/etcd
docker pull k8s.gcr.io/hyperkube:v1.10.13

mkdir -pv data/{etcd0,etcd1,etcd2}

docker-compose up -d
# leave off the "-d" if you want to use
# another terminal tab/window for the next steps

docker exec -it docker-compose-etcd_hyperkube_1 /bin/bash
# you are now in the hyperkube container
kube-apiserver \
  --etcd-servers=http://etcd0:2379 \
  --audit-policy-file=/myaudit/audit-policy.yaml \
  --audit-log-path=/myaudit/audit.log
```
