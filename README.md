
## 📝 Table of Contents

- [About](#)
- [Prerequisites](#)
- [Setup new environment](#)
- [Setup CI/CD on top of Minikube (using helm)](#)
- [Setup build and deploy Toy0store application](#)
- 

## About 
- This is the private mirror repository from [Toy0Store](https://github.com/ahmedmisbah-ole/Devops-Orange.git) application repository.
- These instructions will help to setup new CI/CD tools and configure it to deploy this [repo](https://github.com/ahmedmisbah-ole/Devops-Orange.git).

## Prerequisites
- Ubuntu server 18.04
- At least 4 CPU, 8 GB RAM, and 50 GB HD

## Setup new environment

### 1- Install NFS server:
  > You can Skip it if you already have NFS server 

   - Install nfs packages  `sudo apt install nfs-kernel-server`
   - Create the shared directory `sudo mkdir -p /srv/nfs/kubedata`
   - Change the permission to the new directory `sudo chown nobody:nogroup /srv/nfs/kubedata`
   - Open exports file and add the below line to allow minikube to access the NFS files:
     ```
     sudo vim /etc/exports
     /srv/nfs/kubedata       <minikube Cluster_IP>(rw,sync,no_subtree_check,insecure,no_root_squash,no_all_squash) 
     # <minikube Cluster_IP> could be "*" for  the public access or an range of Ips.
     ```
   - Restart the NFS service `sudo systemctl restart nfs-server`
   - Check that NFS configuration is correct `sudo exportfs -v` 

     > The output should be something like:
     > /srv/nfs/kubedata         <minikube_IP>(rw,wdelay,insecure,no_root_squash,no_subtree_check,sec=sys,rw,insecure,no_root_squash,no_all_squash)
   

### 2- Install local K8s instance using Minikube: 
   
   - Install kubectl
   ```
   apt install conntrack socat
   curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   sudo mv ./kubectl /usr/local/bin/kubectl
   kubectl version --short 
   ```
   - Install Docker
      > some cases you have to install a hypervisor like Qemu or Virtualbox.
   ```
   sudo apt-get update && \sudo apt-get install docker.io -y
   ```
   - Install Minikube   
   ```
   curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
   minikube version
   ```
   - Start Minikube

     * On EC2 without hypervisor 'as a root' run `minikube start --vm-driver=none`
     * General with hypervisor run `minikube start --driver <hypervisor>`
      > minikube start --driver Virtualbox
   - check minikube status `minikube status`
   - get Cluster_IP `kubectl cluster-info`

### 3- install Helm and add tiller:

    ```
    curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
    chmod 700 get_helm.sh
    ./get_helm.sh
    kubectl apply -f helm/rbac.yaml
    helm init --service-account tiller
    kubectl --namespace kube-system get pods | grep tiller
    ```
### 4- deploy the NFS provisioner on minikube:

    - Edit the nfs <NFS_IP> in `manifests/nfs-provisioner/deployment.yaml`
      > in our case it will be the ubuntu server_IP because we installed it locally
    - deploy the provisioner `kubectl apply -f manifests/nfs-provisioner/`

 ## Setup CI/CD on top of Minikube (using helm)

 ###  1- deploy Jenkins:
   - Create new namespace `kubectl create ns build` 
   - Search for jenkins in helm `helm search jenkins`
   - Get the values file  `helm inspect values stable/jenkins > jenkins_values.yaml` or better use the ready file in `helm/helm_values/jenkins_values.yaml`
   - Change the configuration in `helm/helm_values/jenkins_values.yaml` for example:
    
 ```
  serviceType: NodePort
  nodePort: "For Example 32232"
  storageClass: "managed-nfs-storage"
  size: "8Gi"
  plugins:
  - git
  - pipeline
  - CloudBees Docker Build and Publish
  - GitHub
``` 
   - Deploy jenkins with the new values file `helm install stable/jenkins --values helm/helm_values/jenkins_values.yaml --name jenkin --namespace build`
   
- Access jenkins on **http://<cluster_IP>:NodePort/login** to get Jenkins admin password run:
   
`printf $(kubectl get secret --namespace build jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo`
    
### 2- deploy Nexus:
   - Search for nexus in helm `helm search nexus`
   - Get the values file  `helm inspect values stable/sonatype-nexus > nexus_values.yaml` or better use the ready file in `helm/helm_values/nexus_values.yaml`
   - Change the configuration in `helm/helm_values/nexus_values.yaml` for example:
          
   ```
    serviceType: NodePort
    nodePort: "For Example 32233"
    storageClass: "managed-nfs-storage"
    size: "15Gi" 
    nexusDockerHost: <Cluster_IP>
    nexusHttpHost: <Cluster_IP>
   ```
   - Deploy nexus with the new values file:
    
     `helm install stable/sonatype-nexus --values helm/helm_values/nexus_values.yaml --name nexus --namespace build`
      - Access jenkins after waiting 2~3 minutes on **http://<cluster_IP>:NodePort** with default credentials

## Setup build and deploy [Toy0store](https://github.com/ahmedmisbah-ole/Devops-Orange) application
#### - notes:
 - I made a mirror from the Toy0store to make it private in my Github account [link](https://github.com/ahmedbadawy4/Devops-Orange.git) because I added `manifests` for k8s, `Dockerfiles`, CI/CD pipelines, custom `helm` values, NFS provisioner, and the documentation in README.md.

### 1- Build Toy0store Docker images using Jenkins:

   - Open Jenkins UI and create a pipeline Toy0store-build and configure it to get the pipeline script from git repo [link](https://github.com/ahmedbadawy4/Orange-lab-DevOps.git) with script path `./pipelines/build/Jenkinsfile`.
   - Configure Jenkins agents `in my case helm deployed 3 templates [slave, maven, and python]`.
   - Add nexus or Dockerhub credentials.
   - Set the docker images (app and MySQL) version `(it could be dynamic for example ${build_NUMBER} or static)` and the repository URL(Nexus or Docker_hub) in the second stage.
   - Build the pipeline **the images should be pushed to your registry**.
### 2- Deploy Toy0store application in minikube using Ansible:

### 3- Deploy Toy0store application in minikube using Jenkins:
   - Open Jenkins UI and create a pipeline Toy0store-deploy point to the pipeline script in `pipelines/deploy/Jenkinsfile`.
   - Configure the agent access to the target cluster by adding ~/.kube/config and `kubectl` command is installed.
   - set the image tag name for the app and database deployment files. 
      > this step should be automated.
   - Convert database credentials to base64:
     `echo -n 'USERNAME' | base64` and  `echo -n 'PASSWORD' | base64`
   - Create a secret to store database credentials by adding the generated base64 in the below `secret.yaml` 
     
    ```
     apiVersion: v1
     kind: Secret
     metadata:
       name: mysql-secret
     type: Opaque
     data:
       username: <base64 for username>
       password: <base64 for password>
     ```
     
   - Build the pipeline and provide the target `namespace` as a build parameter.
   - Run `kubectl get all -n <target-namespace>` to get the details about the deployment.
   - Access the application **http://<minikube_ip>:30001**
     > an SSL and domain can be added with ingress controller configurations.
## To-DO-List:
  - Adding SSL and domain with ingress controller configurations.
  - Configure Nexus to package and store the java dependencies, jars, and Docker images.
  - Add sonarqube and configure it to test the code and generate reports.
  - Adding a more automatic and efficient strategy to database backup and restore during release deployment (using ansible for example).
