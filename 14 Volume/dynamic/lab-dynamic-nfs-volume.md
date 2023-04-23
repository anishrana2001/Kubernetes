
# Lab for Dynamic NFS volume

## apply the yaml files
```
kubectl apply -f rolebinding.yaml -f storageclass.yaml -f pvc.yaml 
```

## Check the status of StorageClass and PVC
```
[root@master1 volume]# kubectl get sc,pv,pvc
NAME                                     PROVISIONER       RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/nfs-client   k8s-sigs.io/nfs   Delete          Immediate           false                  50s

NAME                               STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/test-claim   Pending                                      nfs-client     50s
```

## Create the Deployment
```
kubectl apply -f dynamic-deployment.yaml 
```

## Again, check the status of StorageClass and PVC. You will observe PV created automatically.
```
[root@master1 volume]# kubectl get sc,pv,pvc
NAME                                     PROVISIONER       RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/nfs-client   k8s-sigs.io/nfs   Delete          Immediate           false                  2m37s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
persistentvolume/pvc-21f33b59-7ab9-4605-ab7f-6dd4c786d4ed   1Mi        RWX            Delete           Bound    default/test-claim   nfs-client              45s

NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/test-claim   Bound    pvc-21f33b59-7ab9-4605-ab7f-6dd4c786d4ed   1Mi        RWX            nfs-client     2m37s
```


## Clear the Lab
```
kubectl delete -f rolebinding.yaml -f storageclass.yaml -f pvc.yaml
rm -f rolebinding.yaml storageclass.yaml pvc.yaml dynamic-deployment.yaml
```

