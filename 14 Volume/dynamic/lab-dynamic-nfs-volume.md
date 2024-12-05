
# Lab for Dynamic NFS volume

## Rolebinding, Storage, PVC and Deployment yaml files

```yaml
cat <<EOF>> dynamic-volume.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: k8s-sigs.io/nfs   # This name must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"
allowVolumeExpansion: true
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
            #image: nginx
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs    # This value must be same in the Storage provisioner
            - name: NFS_SERVER
              value: 192.168.1.34
            - name: NFS_PATH
              value: /home/nfsshare/
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.1.34
            path: /home/nfsshare/
EOF
```
## apply the yaml files
```
kubectl apply -f dynamic-volume.yaml
```

## Check the status of StorageClass and PVC
```
kubectl get sc,pv,pvc
```
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
kubectl get sc,pv,pvc
```
```
[root@master1 volume]# kubectl get sc,pv,pvc
NAME                                     PROVISIONER       RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/nfs-client   k8s-sigs.io/nfs   Delete          Immediate           false                  2m37s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
persistentvolume/pvc-21f33b59-7ab9-4605-ab7f-6dd4c786d4ed   1Mi        RWX            Delete           Bound    default/test-claim   nfs-client              45s

NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/test-claim   Bound    pvc-21f33b59-7ab9-4605-ab7f-6dd4c786d4ed   1Mi        RWX            nfs-client     2m37s
```
## Check the NFS server, you must observe one new directory under /home/nfsshare.

```
[root@server1 nfsshare]# ls -ltr
total 0
drwxrwxrwx. 2 root root 6 Apr 25 22:02 default-test-claim-pvc-4ef753ab-a57a-41d5-aaf7-2eb1dda02df3
[root@server1 nfsshare]#

[root@server1 nfsshare]# cd default-test-claim-pvc-4ef753ab-a57a-41d5-aaf7-2eb1dda02df3/

[root@server1 default-test-claim-pvc-4ef753ab-a57a-41d5-aaf7-2eb1dda02df3]# ls -ltr
total 0

[root@server1 default-test-claim-pvc-4ef753ab-a57a-41d5-aaf7-2eb1dda02df3]# echo "hi" > file.txt

[root@server1 default-test-claim-pvc-4ef753ab-a57a-41d5-aaf7-2eb1dda02df3]# cat file.txt 
hi


```

## Now, delete the PVC, it should remove this newly created directory.
```
kubectl delete pvc/test-claim 
```
```
[root@server1 ~]# cd /home/nfsshare/
[root@server1 nfsshare]# ls -ltr
total 0
```

## It means that, by default reclaim policy is delete and PV will get this value from StorageClass. 
## Clear the Lab
```
kubectl delete -f dynamic-volume.yaml
rm -f dynamic-volume.yaml
```

