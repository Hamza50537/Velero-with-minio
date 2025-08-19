# Kubernetes Cluster Backup and Restore using Velero Example

# Velero

Velero gives you tools to back up and restore your Kubernetes cluster resources and persistent volumes.

# Setup

For my use case i have used the following technology stack.

- Kubernetes Cluster (Kind)
- Velero Command Line Setup
- Minio S3 Compatible Storge
- Install Velero Inside the Cluster
- Deploy a Simple Nginx Application
- Create the Cluster Backup
- View the Cluster Backup on MinIO Portal


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
## Velero Command Line Setup
* Install velero cli that is used to perform the velero operations (it's not a part of Kubernetes cluster):
* [Velero](https://velero.io/docs/v1.8/basic-install/)
* Download the [latest release](https://github.com/vmware-tanzu/velero/releases) tarball for your client platform.
  ```bash
  curl -fsSL https://github.com/vmware-tanzu/velero/releases/latest/download/velero-linux-amd64.tar.gz -o velero.tar.gz
  ```
* Extract the tarball:
  ```bash
  tar -xvf velero.tar.gz
  ```
* Move the extracted velero binary to somewhere in your $PATH (/usr/local/bin for most users).
  ```bash
  cd velero-v1.16.2-linux-amd64
  mv velero /usr/local/bin
  ```
## Deploy MinIO for Backup Storage
* Assuming that you have already cloned the github repo, please follow below steps to deploy minio on local kind cluster.
  ```bash
  # Install git on your platform i am using ubuntu
  apt install git
  # Clone the github repo
  git clone https://github.com/Hamza50537/Velero-with-minio.git
  cd Velero-with-minio
  # Install the minio on kubernetes cluster using this apply command
  kubectl apply -f minio.yaml
  # View the status of minio objects deployed inside the velero namespace
  kubectl get all -n velero
  # Use the port forward command to access the minio ui
  kubectl -n velero port-forward svc/minio 9000:9000 9001:9001
  ```
* Assuming that you have already accessed the minio on your browser using http:/localhost:9001 and logged into the minio using the below credentials.
  * Username: minio
  * Password: minio123
* Create a bucket with the name of velero.

## Deploy Velero into cluster
* Create a file with the name of credentials-velero with following contents
  ```bash
  [default]
  aws_access_key_id=minio
  aws_secret_access_key=minio123
  ```
* Run this command
  ```bash
  velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.7.0 \
  --bucket velero \
  --secret-file ./credentials-velero \
  --use-restic \
  --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://minio.velero.svc.cluster.local:9000 \
  --use-volume-snapshots=false \
  --namespace velero
  # View the status of velero objects deployed inside the velero namespace
  kubectl get all -n velero
  ```
## Deploy nginx sample application inside the cluster
* Assuming that you have already cloned the github repo, please follow below steps to deploy minio on local kind cluster.
  ```bash
  # Install git on your platform i am using ubuntu
  apt install git
  # Clone the github repo
  git clone https://github.com/Hamza50537/Velero-with-minio.git
  cd Velero-with-minio
  # Create the pv in order to store the data
  kubectl apply -f pv.yaml
  # Create the pvc in order to store the data
  kubectl apply -f nginx-pvc.yaml
  # Deploy the nginx application
  kubectl apply -f nginx-deploy.yaml
  # View the status of velero objects deployed inside the velero namespace
  kubectl get all -n nginx
  ```
* As we are using kind cluster which is using docker to run kubernetes inside the container. The pvc data will be stored inside the container and we can verify that by using the following process:

  ```bash
  # Get the container id of the conatianer that is being used to run kind kubernetes cluster 
  docker ps 
  # Go inside the container (Here the container name is kind-control-plane which you can get from the last command)
  docker exec -it kind-control-plane bash
  #  Now go inside the following directory to see the persisted data
  cd mnt/disk1/data/ # You can view the settings of this path in pv.yaml file inside the repo
  ```
## Create the Cluster Backup
* Run the following command to create the backup.
  ```bash
  # Velero command to get the local cluster backup
  velero create backup test
  # You can check the status of the backup
  velero backup describe test
  velero backup logs new
  # You can list the backups
  velero get backups

  ```

## View the Cluster Backup on MinIO Portal
* Run the following command to view the cluster backed up data.
  ```bash
  # Using the port forward command
  kubectl -n velero port-forward svc/minio 9000:9000 9001:9001

  ```
* Assuming that you have already accessed the minio on your browser using http:/localhost:9001 and logged into the minio using the below credentials.
  * Username: minio
  * Password: minio123
* View the newly cluster backup data inside the velero bucket.

