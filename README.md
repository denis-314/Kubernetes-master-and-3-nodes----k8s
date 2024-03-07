Follow the below steps to deploy a Kubernetes CLuster with a master and 3 data nodes on Ubuntu 20.04

ON MASTER NODE
_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

DEPLOYMENT STEPS:
  I. Docker part:

    l. Set up Docker's apt repository (https://docs.docker.com/engine/install/ubuntu/):
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

    2. Install the Docker packages.
        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
     
    3. Verify that the Docker Engine installation is successful by running the hello-world image.
        sudo docker run hello-world
_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________


ON DATA NODES
_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

DEPLOYMENT STEPS:
  I. Docker part:

    l. Set up Docker's apt repository (https://docs.docker.com/engine/install/ubuntu/):
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

    2. Install the Docker packages.
        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
     
    3. Verify that the Docker Engine installation is successful by running the hello-world image.
        sudo docker run hello-world

  II. Kubernetes part:

    1.  Install components versioned 1.28.1 (For version 29 use 1.29.1)
         curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
         echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
         sudo apt update
         sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
         
    2. Disable the swap memory on each server
        sudo swapoff -a

    3. Assign Unique Hostname for Each Server Node
        sudo hostnamectl set-hostname node<number>

    4. Install CRI-O as CRI Container Runtime endpoint: https://computingforgeeks.com/install-cri-o-container-runtime-on-ubuntu-linux/
        OS=xUbuntu_20.04
        CRIO_VERSION=1.23
        echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
        echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$CRIO_VERSION/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list
		
        curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION/$OS/Release.key | sudo apt-key add -
        curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add -
		
        sudo apt update
        sudo apt install cri-o cri-o-runc
		
        apt show cri-o
        sudo systemctl enable crio.service
        sudo systemctl start crio.service
        systemctl status crio

    5. Join the Worker Node to Cluster
         sudo kubeadm join 10.40.0.124:6443  --cri-socket=unix:///var/run/crio/crio.sock --token ge6gm6.pf6i6wo72g2aqx0g \--discovery-token-ca-cert-hash sha256:b6503b53df736ac718e74da234fd82628786f7e2df04b5fcbd291b3b0a6bca1b

         Obs! replace token and cert
           sudo kubeadm join 10.40.0.124:6443  --cri-socket=unix:///var/run/crio/crio.sock --token <token_generated_on_the_master> \--discovery-token-ca-cert-hash <key_generated_on_the_master>
_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________


IMPORTANT !

After a VM reboot, it is possible that Kubernetes cluster to be no longer available, reporting the following error:
	”The connection to the server <node
