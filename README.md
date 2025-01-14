# Complete CICD with Jenkins,Maven,SonarQube,Docker,Kubernetes and ArgoCD

## CI Setup

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

### 6. Create pipeline using [Jenkinsfile](https://github.com/anuja2015/CICDwithArgo/blob/master/sourcecode/Jenkinsfile)

## Note: Make sure to add inbound rules in NSG for azure VM network to allow traffic on 3010(springboot app) and 9000 (sonarqube) port.







      
    
