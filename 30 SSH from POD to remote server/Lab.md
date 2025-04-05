- [] # **How to perform SSH from POD to remote server?**
---
- [] >  💡**Note**:
> ### Local server IP 192.168.1.30, this server is the part of Kubernetes 
>  ### Remote Host "192.168.1.30", this server is not the part of Kubernetes cluster.


### Below are the Steps, that need to be done.
---
+ ### <mark> Step 1. Create an image which has SSH Package.</mark>
+ ### <mark> Step 2. Push the image on DockerHub repo.</mark>
+ ### <mark> Step 3. Create a pod with using our own customized image.</mark>
+ ### <mark> Step 4. Create a RSA Key and copy the public key on a remote server.</mark>
+ ### <mark> Step 5. Execute the "ssh" command, and it should not ask for a password.</mark>

- []
> ## Prerequise: 🙂
---
### 1. ".ssh" directory must be created on a Remote server.
### 2. Local server must be updated.

> ### Solution:
---
### Prerequise: 1
### Check the remote server, ".ssh" directory.
```
ls -ltr /home/arana/.ssh/
```

### Prerequise 2: 
### Now, login into the local server .i.e. 192.168.1.30
```
yum update -y
```
### <mark> Step 1. Create an image which has SSH Package.</mark>
```

cat <<EOF>> Dockerfile
# Use the official Ubuntu image as the base image
FROM ubuntu:latest
# Update mirrors and clean package lists
RUN sed -i 's/archive.ubuntu.com/mirrors.edge.kernel.org/' /etc/apt/sources.list && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && apt-get update --fix-missing
# Update package lists and install OpenSSH server
RUN apt-get update -y && \
    apt-get install -y openssh-server && \
    mkdir /var/run/sshd

# Set root password (change "rootpassword" to a secure password)
RUN echo 'root:rootpassword' | chpasswd

# Permit root login via SSH
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# Expose port 22 for SSH
EXPOSE 22

# Start the OpenSSH server
CMD ["/usr/sbin/sshd", "-D"]
EOF
```
### Let's verify if any images are there?

```
docker image ls
```
### Create the image
```
docker image build -t ssh-server .
```
### A new image must be created. 
```
docker image ls
```
### Let's login into the container.
```
docker container run -it --name mynewcontainer1 ssh-server /bin/bash
```
#### Go to "cd /root/.ssh" directory
```
cd /root/.ssh/
```
#### Create the key and then copy the key into the remote server.
```
ssh-keygen -t rsa
```

```
ssh-copy-id arana@192.168.1.30
```
#### Now, try if we can able to login into the remote server?
```
ssh arana@192.168.1.30
```

### Exist from the container.

```
docker image ls
```
### <mark> Step 2. Push the image on DockerHub repo.</mark>
#### Login into the Docker, so that we can push the image to our repo.
```
docker login
```
#### Tag the image.
```
docker tag ssh-server anishrana2001/ssh-server:latest
```
#### Verify it again.
```
docker image ls
```
#### Check the docker image in DockerHub
```
https://hub.docker.com/repositories/anishrana2001
```

#### Now, push the image to your Docker repo.
```
docker push anishrana2001/ssh-server
```

#### Perform the post check.

```
https://hub.docker.com/repositories/anishrana2001
```

### <mark> Step 3. Create a pod with using our own customized image.</mark>
### Now, its time to create a pod.
```
kubectl run ssh-pod --image=anishrana2001/ssh-server:latest
```
#### Check the pod is running ?
```
kubectl get pods -w
```
#### Login into our POD.
```
kubectl exec -it pods/ssh-pod -- /bin/bash
```
#### Go to the .ssh directory.
```
cd /root/.ssh/
```
### <mark> Step 4. Create a RSA Key and copy the public key on a remote server.</mark>
#### Generate the Key
```
ssh-keygen -t rsa
```
####  Copy the key into our remote server.
```
ssh-copy-id arana@192.168.1.30
```
### <mark> Step 5. Execute the "ssh" command, and it should not ask for a password.</mark>
#### Execute the Post checks. 
```
ssh arana@192.168.1.30
```
