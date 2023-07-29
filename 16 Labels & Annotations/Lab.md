# Labels

## How check the label for object?
### Create a pod for testing.
```
kubectl run nginx-pod --image=nginx 
```
```
kubectl get pods nginx-pod --show-labels
```
```
kubectl label pods nginx-pod release=stable
```
```
kubectl get pods nginx-pod --show-labels
```

## How to update the labels on existing pods?
```
kubectl label pods nginx-pod release=canary --overwrite
```
```
kubectl get pods nginx-pod --show-labels
```
```
kubectl get deploy -l "environment=production"
```
## How to add the label on manifest file?
```
cat <<EOF>> label-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: env-deploy
  labels:
    app: env-deploy
    release: stable
    partition: customerA
spec:
 replicas: 3 
 selector:
    matchLabels:
        app: env-deploy
    matchExpressions:
     - { key: release, operator: In, values: [stable] }
     - { key: partition, operator: In, values: [customerA] }
     - { key: environment, operator: NotIn, values: [dev] }  
 template:
    metadata:
      labels:
        app: env-deploy
        release: stable
        partition: customerA
    spec:
      containers:
      - name: env-cont
        image: nginx
EOF
```
```
kubectl create -f label-deploy.yaml
```

```
kubectl get deploy --show-labels
```
```
kubectl get deploy –l release=stable
```
### You can also check the pods
```
kubectl get pods –l release=stable
```
```
kubectl get deploy -l release=stable --output=wide
```
```
kubectl get pods -l -n A
```

### How to delete the labels from object?
```
kubectl label nginx-pod release-
```
```
kubectl get pods nginx-pod --show-labels
```

## How to add the label on nodes?
### How to check the labels for node?
```
kubectl get nodes --show-labels
```
### Command syntax to add the label on node.
```
kubectl label nodes <your-node-name> disktype=ssd
```

```
kubectl label nodes workernode1.example.com disktype=ssd
```
### Verify if label is correctly set on node.
```
kubectl get nodes --show-labels
```
### Question: You have been asked to schedule a pod on node which has label set for "disktype=ssd"

### In our case, workernode1 has this label.
```
cat <<EOF>> nodepod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:      # Line need to be added 
    disktype: ssd    # Line need to be added
EOF
```
```
kubectl create -f nodepod.yaml
```
```
kubectl get pods --output=wide
```

# Annotations
## How to add the Annotations?

### Create a deployment 
```
kubectl create deployment nginx --image=nginx --replicas=3
```
```
kubectl get deployments.apps 
```
### How to check the annotations from object metadata?
```
kubectl get  deployments.apps nginx -o=json | jq .metadata.annotations
```
### We can also use this "jsonpath" for identify the annotations.
```
kubectl get  deployments.apps nginx -o=jsonpath="{.metadata.annotations}" | jq
```
### Annotations add with the help of "kubectl annotate" command.
```
kubectl annotate   deployment nginx "application=Nginx-web-server"
```
### Check the annotations
```
kubectl get  deployments.apps nginx -o=json | jq .metadata.annotations
```

### Annotations is not added at pod template level. Let's check the at pod level.
```
kubectl get  deployments.apps nginx -o=json | jq  .spec.template.metadata.annotations
```
### Adding annotations at pod level.
```
kubectl  patch   deployment nginx -p '{"spec": {"template":{"metadata":{"annotations":{"pod/osversion":"12.0"}}}} }'
```
### We can verify if annotation is added correctly at pod level.
```
kubectl get  deployments.apps nginx -o=json | jq .spec.template.metadata.annotations
```

## How to add the annotations with "kubectl run" command?
```
kubectl   run   webpod --image=nginx    --annotations="OS/version=8.5"
```
```
kubectl get pods webpod -o=json | jq .metadata.annotations
```

### How to delete the Annotations?

#### How to delete the annotations from Object metadata?
```
kubectl get  deployments.apps nginx    -o=json | jq .metadata.annotations
```
```
kubectl annotate deployments.apps nginx application-
```
```
kubectl patch deployments/nginx --type=json -p='[{"op": "remove", "path": "/metadata/annotations/application"}]'
```
```
kubectl get  deployments.apps nginx -o=json | jq .metadata.annotations
```

#### How to delete the annotations from Pod template?
```
kubectl get  deployments.apps nginx -o=json | jq  .spec.template.metadata.annotations
```
```
kubectl patch deployments/nginx --type=json -p='[{"op": "remove", "path": "/spec/template/metadata/annotations/pod~1osversion"}]'
```
```
kubectl get  deployments.apps nginx -o=json | jq  .spec.template.metadata.annotations
```
