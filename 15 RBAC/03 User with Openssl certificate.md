
## Generate Private Key

### Step 1. Generate Private Key
####  - The first step in creating a client certificate for a user is to generate a private key. 
####  - This Private key will be used to sing the certificate and must be kept secure.
####  - In order to generate private key, we can use Openssl,  easyrsa, cfssl. I am using Openssl tool. 

```
openssl genrsa -out raja.key 2048
```
#### Above command generate the 2048 Bit RSA private key and save it on the file raja.key.

### Step 2.  Create a certificate signing request (CSR)

#### - In this step, we are going to create a CSR with the help  of private key that we generated at Step 1.
####  - The CSR contains information about the user, for an example, it's name and email address, and is used to request a digital certificate from a certificate authority (CA).
```
openssl req -new -key raja.key -out raja.csr -subj "/CN=raja/O=dev/O=example.org"
```

#### In the above command, "-subj" flag specifies the subject of the CSR, which include "CN" (Command Name", and "O" (Organization).However, in the Kubernetes, it use "O" as a group name. 
##### - Command Name is Raja 
####  - This will be used to identify him against the API Server.
#####  - Group (O) = dev
##### - Group (O) = example.org
#### - Above command will create another file called "raja.csr"

### Step 3.  Sign the certificate with a CA (certificate authority)
#### When we installed the Kubernetes cluster with kubeadm command, at that time, CA also installed. 
####  A CA is a trusted entity that issues digital certificates, which are used to authenticate users and services.
#### In production environment, we may use public CA, such as DigiCert. 
#### Here, I am going to use inbuilt CA to self-sign CA with Openssl.
#### By default, CA path would be "/etc/kubernetes/pki/"
```
openssl x509 -req -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -days 730 -in raja.csr -out raja.crt
```
#### In the above command, 
#### CA certificate is "/etc/kubernetes/pki/ca.crt"
#### CAkey is "/etc/kubernetes/pki/ca.key"
#### CSR = raja.csr
#### CA will generate the certificate file in "raja.crt"


#### Check the Kubernetes config file 
```
kubectl config view
```
### Step 4. Generate the kubeconfig file for this user. 
```
kubectl config set-credentials raja --client-certificate=/root/rbac/raja.crt --client-key=/root/rbac/raja.key --embed-certs=true
```

#### I am using user certificate & Key path i.e. "/root/rbac/", in your case it may be different. 
#### Check the Kubernetes config file 
```
kubectl config view --raw 
```
## Check the current context in Kubernetes cluster

```
kubectl config get-contexts
```

#### Step 5. Create context for raja user.

```
kubectl config set-context raja-context --cluster=kubernetes  --namespace=core --user=raja 
```
## Check the context in Kubernetes cluster again. You will notice one more context.

```
kubectl config get-contexts
```

### Check the current context 
```
kubectl config current-context 
```
### How to change the context for raja user?
```
kubectl config use-context raja-context
```

### Check the current context 
```
kubectl config current-context 
```

#### Multiple way to check if user has privileges to access the resources
```
kubectl --context=raja-context get pods
```
```
kubectl get pods -n core
```
```
kubectl get pods 
```
#### Change the context to Admin again.
```
kubectl config use-context kubernetes-admin@kubernetes
```
```
kubectl get pods -n core --as raja
```
```
kubectl auth can-i list pods --namespace core --as raja
```
```
kubectl auth can-i list secrets --namespace core --as raja
```
