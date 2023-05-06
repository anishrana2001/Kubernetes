
## How to create the configMap from command line?
```
kubectl create configmap command-configmap --from-literal=NGINX_PORT=8080 --from-literal=language=english --from-literal=environments=production,development,test
```

## How to clear the lab.
```
kubectl delete configmap command-configmap
```

 
 
 
 
 
 
 
 
 
 
