
## Disk CSI-Plugin

An Disk CSI plugin is available to help simplify storage management.
Once user creates PVC with the reference to a Disk storage class, disk and
corresponding PV object gets dynamically created and becomes ready to be used by
workloads.

## Features Support

**Disk Snapshot:** [disk-snapshot](./disk-snapshot.md)

**Block Volumes:** [disk-block](./disk-block.md)

**Shared Disk:** [disk-shared](./disk-shared.md)

**Volume Attach Limits:** [volume-attach-limits](./disk-volume-limits.md)

## Configuration Requirements

* Secret object with the authentication (Access Key ID/Secret) and privilege to buy/attach ETC and disks.
* StorageClass with diskplugin (default diskplugin.csi.alibabacloud.com name) as a provisioner name and information about disk(zoneId, regionId, type)
* Service Accounts with required RBAC permissions


## How to Use

### Requirements

1. A kubernete cluster on aliyun ETC.
2. Ensure service account admin under namespace kube-system

```
kubectl apply -f ./deploy/rbac.yaml

```

### Step 1: Create CSI disk-plugin
If the cluster is not in STS mode, ACCESS_KEY_ID and ACCESS_KEY_SECRET can be set as environment variable via secret object. 

check `deploy/disk/disk-plugin.yaml` and `deploy/disk/disk-provision.yaml`. 

If only for a quick test, key and secret can be filled in as well.


```
# kubectl create -f ./deploy/disk/disk-plugin.yaml
```

> Note: The plugin log style can be configured by environment variable: LOG_TYPE.

> "host": logs will be printed into files which save to host(/var/log/alicloud/diskplugin.csi.alibabacloud.com.log);

> "stdout": logs will be printed to stdout, can be printed by docker logs or kubectl logs.

> "both": default option, log will be printed both to stdout and host file.

### Step 2: Create CSI disk-provisioner
```
# kubectl create -f ./deploy/disk/disk-provisioner.yaml
```

### Step 3: Create StorageClass

```
# kubectl create -f ./deploy/disk/storageclass.yaml
```

**Important:** storageclass.yaml, must be customized to match your environment: zoneId, regionId;

### Step 4: Added PVC

```
# kubectl create -f ./deploy/disk/disk-pvc.yaml
```

After successfully finished this step, an unattached 25G ssd disk can be found in aliyun ECS dashboard. 


**Check status of csi plugins and provisiones.**

```
# kubectl get pods | grep csi
```

The following output should be displayed:

```
# kubectl get pod
NAME                    READY   STATUS    RESTARTS   AGE
csi-disk-provisioner-0  2/2     Running   0          7s
csi-disk-plugin-4hg7v   2/2     Running   0          3s
csi-disk-plugin-rhjp9   2/2     Running   0          3s
csi-disk-plugin-x229t   2/2     Running   0          3s
```

**Check PVC**

```
# kubectl get pvc | grep disk-pvc
```

A newly created pvc with BOUND status should exist.

if no such a pvc or the status is not bound, go to describe pvc and find more details.


### Step 5: Mount pvc to a pod

```
# kubectl create -f ./deploy/disk/example_deploy.yaml
```

**Check if pod has a running status**

```
# kubectl get pod
NAME                                 READY     STATUS    RESTARTS   AGE
nginx-deployment1-5879d9db88-49n8m   1/1       Running   0          37m
```

And that's all.
