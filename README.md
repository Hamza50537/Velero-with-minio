# Kubernetes Cluster Backup and Restore using Velero Example

# Velero

Velero gives you tools to back up and restore your Kubernetes cluster resources and persistent volumes.

# Setup

For my use case i have used the following technology stack.

- Kubernetes Cluster (Kind)
- Minio s3 Compatible Storge
- Velero 


## Kubernetes Cluster Setup
* Install docker as per your operating system. I have used ubuntu:
  * [Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

#To install the latest version
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```
* Install the kubectl for interacting with the kubernetes cluster
  * [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
```bash
   # For x86-64 architecture
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   # For ARM64 architecture
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl"
   #Install kubectl
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
* Install the kind cluster
  * [Kind](https://kind.sigs.k8s.io)
```bash
   # For AMD64 / x86_64
   [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-amd64
   # For ARM64
   [ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-arm64
   chmod +x ./kind
   sudo mv ./kind /usr/local/bin/kind
   
   # Create kind cluster
   kind create cluster # Default cluster context name is `kind`.
```

[MIT](https://choosealicense.com/licenses/mit/)
