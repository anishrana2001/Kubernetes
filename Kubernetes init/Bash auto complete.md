
## How to autocomplete the commands for kubernetes ? **


```
yum install bash-completion
```
```
source  /usr/share/bash-completion/bash_completion
```
```
echo 'source <(kubectl completion bash)' >>~/.bashrc
```
```
kubectl completion bash > /etc/bash_completion.d/kubectl
```
## Logout and Login again.