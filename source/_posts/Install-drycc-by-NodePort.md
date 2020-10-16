---
title: Install drycc by NodePort and aliyun oss
date: 2019-03-15 17:39:43
tags:
---

```
helm install \
  --name drycc \
  --namespace drycc \
  --set global.storage=oss \
  --set router.service.type=NodePort \
  --set router.service.nodePort.builder=30022 \
  --set router.service.nodePort.http=30080 \
  --set router.service.nodePort.https=30443 \
  --set router.service.nodePort.healthz=30090 \
  --set oss.accesskey="xxxx" \
  --set oss.secretkey="xxxxxxxxxxxxxxxxxxxxxx" \
  --set oss.endpoint="https://oss-cn-beijing.aliyuncs.com" \
  --set oss.builder_bucket="drycc-builder" \
  --set oss.registry_bucket="drycc-registry" \
  --set oss.database_bucket="drycc-database" \
  drycc/workflow
```
