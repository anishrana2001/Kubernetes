

### How to clear the lab?

```
kubectl delete statefulsets.apps nginx-sts 
kubectl delete persistentvolumeclaim/pg-pvc-nginx-sts-0 persistentvolumeclaim/pg-pvc-nginx-sts-1 persistentvolumeclaim/pg-pvc-nginx-sts-2 persistentvolume/pv-sts-0 persistentvolume/pv-sts-1 persistentvolume/pv-sts-2 service/svc-nginx-sts
```
