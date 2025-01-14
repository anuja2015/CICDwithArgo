# Complete CICD with Jenkins,Maven,SonarQube,Docker,Kubernetes and ArgoCD

## Set up Jenkins as a docker container.

    1. Create a virtual machine on Azure. I created a Ubuntu 22.04 VM of size Standard D4s v3 (4 vcpus, 16 GiB memory).
    
    2. Install Docker on VM.

          ### Set up Docker's apt repository.

          #### Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

Install latest version

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


      
    
