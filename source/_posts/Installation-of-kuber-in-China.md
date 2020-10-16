---
title: Installation of kuber in China
date: 2019-04-03 17:20:04
tags:
---

# Requirements

This paper takes [kubespray]() as an example, other approaches are similar.

- A cloud host in Hong Kong or other non-mainland China
- Cloud hosts need to install docker

# Install kubespray and generate inventory

```
git clone https://github.com/kubernetes-sigs/kubespray
cd kubespray && python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt -i https://pypi.douban.com/simple
cp -rfp inventory/sample inventory/drycc
declare -a IPS=(10.1.80.51 10.1.80.52 10.1.80.53 10.1.80.54)
CONFIG_FILE=inventory/drycc/hosts.ini python3 contrib/inventory_builder/inventory.py
```

# Modify the `inventory/drycc/group_vars/all/docker.yml` file

```
docker_insecure_registries:
  - gcr.io
  - quay.io

docker_registry_mirrors:
  - https://registry.docker-cn.com

```

# Start proxy and docker mirrors

This step needs to be performed on cloud hosts in non-mainland China. Let's assume that his IP is `47.15.11.11`.

```
git clone https://github.com/duanhongyi/docker-mirrors
cd docker-mirrors
./start.sh
```

# DNS hijacking

Let's take modifying `/etc/hosts` as an example and hijack DNS to this cloud host.

```
47.52.172.207 gcr.io quay.io
```

That's good. You can install kubernetes happily.

```
ansible-playbook -i inventory/drycc/hosts.ini --become --become-user=root cluster.yml
```

# Other options

After installation, you can also hijack the DNS of kubernetes to the cloud host.

Take coredns as an example:

```
kubectl edit ConfigMap/coredns -n kube-system
```

Modify the data item:

```
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream /etc/resolv.conf
          fallthrough in-addr.arpa ip6.arpa
        }
        hosts gfw.drycc.cc {
          47.15.11.11 gfw.drycc.cc
          fallthrough
        }
        rewrite name regex gcr\.io gfw.drycc.cc
        rewrite name regex quay\.io gfw.drycc.cc
        rewrite name regex github\.com gfw.drycc.cc
        rewrite name regex (.*)\.amazonaws\.com gfw.drycc.cc
        rewrite name regex (.*)\.s3\.amazonaws\.com gfw.drycc.cc
        rewrite name regex (.*)\.googleapis\.com gfw.drycc.cc
        prometheus :9153
        proxy . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
```
