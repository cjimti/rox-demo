# ReadOnlyMany Demo

Steps to create a ReadOnlyMany (ROX) PVC on Kubernetes:
- Create a RWO (ReadWriteOnce) PVC
- Use a Pod to mount and populate the RWO PVC then delete the Pod.
- Create a ROX (ReadOnlyMany) PVC from the existing RWO (ReadWriteOnce) PVC
- Mount the new ROX PVC with a Pod.
- Delete the original RWO PVC
- Additional Pods may now mount the new ROX PVC.

## Steps

Create `pvci-test` Namespace:
```shell
kubectl apply -f ./100-ns-rox-demo.yml
```

Create a ReadWriteOnce PVC `demo-rwo`:
```shell
kubectl apply -f ./200-pvc-demo-rwo.yml
```

List PVCs in the `pvci-test` namespace:
```shell
kubectl get pvc -n rox-demo
```

Example output:
```plain
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
demo-rwo   Bound    pvc-770e857c-490b-4899-be3b-0a54ad1276a3   100Mi      RWO            rook-ceph-block   17s
```

Attach a Pod to the `demo-rwo` in order to populate the PVC with a file:
```shell
kubectl apply -f ./300-pod-populate.yml
```

Review PVC mount in Pod:
```shell
kubectl exec populate -n rox-demo -- df -m |grep /data
```

Example output:
```plain
Defaulted container "populate" out of: populate, init-populate (init)
/dev/rbd8                   93        82         9  90% /data
```

Review populated file in Pod:
```shell
kubectl exec populate -n rox-demo -- ls -lh ./data
```

Example output:
```plain
Defaulted container "populate" out of: populate, init-populate (init)
total 81933
-rw-r--r--    1 1000     2000       80.0M Mar 16 21:23 80mb.txt
drwxrws---    2 root     2000       12.0K Mar 16 21:23 lost+found
```

Delete Pod:
```shell
kubectl delete -f ./300-pod-populate.yml
```

Create a ROX (ReadOnlyMany) PVC from the existing `demo-rwo` RWO (ReadWriteOnce) PVC:
```shell
kubectl apply -f ./400-pvc-demo-rox.yml
```

Review existing PVCs:
```shell
kubectl get pvc -n rox-demo
```

Example output:
```plain
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
demo-rox   Bound    pvc-f98cd5ce-34ac-4b84-bd46-5e8b51802345   100Mi      ROX            rook-ceph-block   4s
demo-rwo   Bound    pvc-e210934a-f445-4f62-a214-70043b36b28a   100Mi      RWO            rook-ceph-block   10m
```

Create a POD mounted to the new `demo-rox`:
```shell
kubectl apply -f ./500-pod-rox-mount-a.yml
```

Review file on new `rox-mount-a` Pod:
```shell
kubectl exec rox-mount-a -n rox-demo -- ls -lh ./data
```

Delete original `demo-rwo` (ReadWriteOnce) PVC:
```shell
kubectl delete pvc demo-rwo -n rox-demo
```

Create another POD mounted to the new `demo-rox`:
```shell
kubectl apply -f ./600-pod-rox-mount-b.yml
```

Review file on new `rox-mount-b` Pod:
```shell
kubectl exec rox-mount-b -n rox-demo -- ls -lh ./data
```

Review **Used By:** section of the ROX PVC:
```shell
kubectl describe pvc demo-rox -n rox-demo
```