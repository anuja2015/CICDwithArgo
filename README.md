# Complete CICD with Jenkins,Maven,SonarQube,Docker,Kubernetes and ArgoCD

## __CI Setup__

### 1. Create a virtual machine on Azure. I created a Ubuntu 22.04 VM of size Standard D4s v3 (4 vcpus, 16 GiB memory).

### 2. Install Docker on VM.

#### Set up Docker's apt repository.

##### Add Docker's official GPG key:
            sudo apt-get update
            sudo apt-get install ca-certificates curl
            sudo install -m 0755 -d /etc/apt/keyrings
            sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
            sudo chmod a+r /etc/apt/keyrings/docker.asc

##### Add the repository to Apt sources:
            echo \
              "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
              $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
                sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
            sudo apt-get update

##### Install latest version

            sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

### 3. Manage Docker as a non-root user

#### Create the docker group.

             sudo groupadd docker

#### Add your user to the docker group.

             sudo usermod -aG docker $USER

##### Log out and log back in so that your group membership is re-evaluated.

##### If you're running Linux in a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.

##### You can also run the following command to activate the changes to groups:

             newgrp docker

##### Verify that you can run docker commands without sudo.

### 4. Set up Jenkins as docker container.

##### Using the [Dockerfile](https://github.com/anuja2015/CICDwithArgo/blob/master/jenkins/Dockerfile) create a custom jenkins image.

            docker build -t armdevu/custom-jenkins:1.0 .

            docker run -p 8080:8080 -p 50000:50000 -d --name myjenkins -u root -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock armdevu/custom-jenkins:1.0

#### Get Initial admin password

            docker exec -it myjenkins /bin/bash
            
            jenkins@4b98fded05ff:/$ cat /var/jenkins_home/secrets/initialAdminPassword

## Note: Make sure to add inbound rules in NSG for azure VM network to allow traffic on 8080 port.

### 5. Setup a pipeline agent as docker container.

##### Use the [Dockerfile](https://github.com/anuja2015/CICDwithArgo/blob/master/agent/Dockerfile) to create custom image for the pipeline agent. The image will have maven, docker and openjdk-17 installed.

### 6. Setup Sonarqube server.

#### 1. Install Sonar scanner plugin on jenkins.

#### 2. Download and Install sonarqube server on the azure VM where jenkins is running.

##### Add a used sonarqube

            azureuser@JumpServer:~$ sudo su -
            root@JumpServer:~# adduser sonarqube
            Adding user `sonarqube' ...
            Adding new group `sonarqube' (1001) ...
            Adding new user `sonarqube' (1001) with group `sonarqube' ...
            Creating home directory `/home/sonarqube' ...
            Copying files from `/etc/skel' ...
            New password:
            Retype new password:
            passwd: password updated successfully
            Changing the user information for sonarqube
            Enter the new value, or press ENTER for the default
                    Full Name []:
                    Room Number []:
                    Work Phone []:
                    Home Phone []:
                    Other []:
            Is the information correct? [Y/n] Y

##### Install unzip
            root@JumpServer:~# apt install unzip
            Reading package lists... Done
            Building dependency tree... Done
            Reading state information... Done
            Suggested packages:
              zip
            The following NEW packages will be installed:
              unzip
           
            
##### Download the sonarqube installation binaries

            azureuser@JumpServer:~$ sudo su - sonarqube
            sonarqube@JumpServer:~# wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.1.0.102122.zip
            --2025-01-14 16:53:56--  https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.1.0.102122.zip
            Resolving binaries.sonarsource.com (binaries.sonarsource.com)... 99.84.188.21, 99.84.188.35, 99.84.188.106, ...
            Connecting to binaries.sonarsource.com (binaries.sonarsource.com)|99.84.188.21|:443... connected.
            HTTP request sent, awaiting response... 200 OK
            Length: 825789639 (788M) [binary/octet-stream]
            Saving to: ‘sonarqube-25.1.0.102122.zip’

            sonarqube-25.1.0.102122.zip                100%[========================================================================================>] 787.53M  73.9MB/s    in 10s

            2025-01-14 16:54:06 (75.0 MB/s) - ‘sonarqube-25.1.0.102122.zip’ saved [825789639/825789639]


##### Unzip the binaries.

            sonarqube@JumpServer:~# unzip sonarqube-25.1.0.102122.zip
            Archive:  sonarqube-25.1.0.102122.zip

##### Change the permissions and ownership

            sonarqube@JumpServer:~$ chmod -R 755 /home/sonarqube/sonarqube-25.1.0.102122
            sonarqube@JumpServer:~$ chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-25.1.0.102122

##### Start sonarqube server depending on the os distribution

            cd /home/sonarqube/sonarqube-25.1.0.102122/bin
            
            sonarqube@JumpServer:~/sonarqube-25.1.0.102122/bin$ cd linux-x86-64
            
            sonarqube@JumpServer:~/sonarqube-25.1.0.102122/bin/linux-x86-64$ ls -lrt
            total 8
            -rwxr-xr-x 1 sonarqube sonarqube 7195 Jan  7 10:30 sonar.sh
            sonarqube@JumpServer:~/sonarqube-25.1.0.102122/bin/linux-x86-64$ ./sonar.sh start
            /usr/bin/java
            Starting SonarQube...
            Started SonarQube.
            
##### Access sonarqube at <ipaddress-of-the-VM>:9000. Username: admin Initial password: admin.

## Note: Make sure to add inbound rules in NSG for azure VM network to allow traffic on  9000 (sonarqube) port.

![Screenshot 2025-01-14 225040](https://github.com/user-attachments/assets/e4fd78df-9e10-42e1-903e-bcfd8a7075e3)
![Screenshot 2025-01-14 225307](https://github.com/user-attachments/assets/241faf27-2651-49cd-a765-c9da6af32bab)

### 7. Add sonarqube credentials in jenkins

#### Create token on sonarqube server for jenkins authentication

![Screenshot 2025-01-14 225717](https://github.com/user-attachments/assets/a9ab7798-094b-4e15-8097-07c0331cfc40)

#### Add the token to jenkins credentials

![Screenshot 2025-01-14 225825](https://github.com/user-attachments/assets/a8d19bab-23a8-42cb-b20a-6df136f2ffef)

### 8. Create pipeline using [Jenkinsfile](https://github.com/anuja2015/CICDwithArgo/blob/master/sourcecode/Jenkinsfile)

## Note: Make sure to add inbound rules in NSG for azure VM network to allow traffic on 3010(springboot app)  port.

![Screenshot 2025-01-14 231154](https://github.com/user-attachments/assets/a3792491-d7a2-4dc4-8c34-41808626d59a)


## __CD Setup__

### 1. Create AKS cluster

##### I created AKS cluster using Terraform [Terraform files](https://github.com/anuja2015/devopsquestionare2024/tree/master/AKSTerraform)

### 2. Configure kubeconfig

            $ az aks get-credentials --name argocd-k8s --resource-group aks-resource-group
            Merged "argocd-k8s" as current context in /home/armdevu/.kube/config

### 3. Install ArgoCD

            ~$ kubectl create ns argo-cd
               namespace/argo-cd created
            $ kubectl apply -n argo-cd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


### 4. Configure ArgoCD

##### Get services under the namespace argo-cd

            $ kubectl get service -n argo-cd           
            NAME                                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE              argocd-applicationset-controller          ClusterIP   10.0.98.42     <none>        7000/TCP,8080/TCP            8m12s
            argocd-dex-server                         ClusterIP   10.0.118.7     <none>        5556/TCP,5557/TCP,5558/TCP   8m12s
            argocd-metrics                            ClusterIP   10.0.40.252    <none>        8082/TCP                     8m11s
            argocd-notifications-controller-metrics   ClusterIP   10.0.12.134    <none>        9001/TCP                     8m10s
            argocd-redis                              ClusterIP   10.0.216.140   <none>        6379/TCP                     8m9s
            argocd-repo-server                        ClusterIP   10.0.169.237   <none>        8081/TCP,8084/TCP            8m8s
            argocd-server                             ClusterIP   10.0.43.40     <none>        80/TCP,443/TCP               8m7s
            argocd-server-metrics                     ClusterIP   10.0.162.187   <none>        8083/TCP                     8m6s

##### The service we require to access ArgoCD is argocd-server. To access this we have to change the service type to LoadBalancer(AKS) or NodePort(minikube).

            $ kubectl edit service argocd-server -n argo-cd
            service/argocd-server edited
            $ kubectl get service -n argo-cd | grep argocd-server
            argocd-server                             LoadBalancer   10.0.43.40     172.171.55.105   80:31322/TCP,443:30660/TCP   14m
            argocd-server-metrics                     ClusterIP      10.0.162.187   <none>           8083/TCP                     14m

#####



      
    
