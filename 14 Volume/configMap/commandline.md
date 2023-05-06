
## How to create the configMap from command line?
```
kubectl create configmap command-configmap --from-literal=NGINX_PORT=8080 --from-literal=language=english --from-literal=environments=production,development,test
```

## How to clear the lab.
```
kubectl delete configmap command-configmap
```
## Question: 

You are tasked to create a ConfigMap and consume the ConfigMap in a pod using a volume mount. 
Please complete the following.
- Create a ConfigMAP named another-config containing the key/value pare. Key3/value2.
- Start a pod named nginx-configmap containing a single container using the nginx image.
- and mount the key you just created into the pod under directory /some/path


```
kubectl create configmap another-config --from-literal=key3=value2
```
## Create a POD, name=nginx-configmap ; image: nginx ; mountPath: "/some/path" ; configMap: name: another-confi
```
cat <<EOF>>question-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-configmap
spec:
  containers:
  - name: mypod
    image: nginx
    volumeMounts:
    - name: foo
      mountPath: "/some/path"
  volumes:
  - name: foo
    configMap:
      name: another-config
 EOF
 ```
 
 ```
 kubectl create -f question-configmap.yaml
 ```
 
 ## Check if file is correctly mounted on the container?
 
 ```
 kubectl exec -it nginx-configmap -- ls -l /some/path
 ```
 ## Open the file and verify the value.
 ```
 kubectl exec -it nginx-configmap -- cat /some/path/key3 ; echo 
 ```
 
 
 ## How to clear the lab + question?
 
 ```
 kubectl delete pod/nginx-configmap  --grace-period=0 --force
 kubectl delete  configmaps another-config
 rm -f question-configmap.yaml
 ```
 
 
 
 
 
 
 
 
 
 
