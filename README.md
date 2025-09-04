# Deploying triorasystems application in k8s(kubeadm).

# kubeadm installation
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl

    # Enable required kernel modules
    sudo modprobe overlay
    sudo modprobe br_netfilter

    # Set sysctl params required by Kubernetes
    cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF
    
    sudo sysctl --system

    # Install container runtime (containerd recommended) Kubeadm doesn’t install a container runtime by default. Install and configure containerd:
    sudo apt-get update
    sudo apt-get install -y containerd
    
    # Generate default config:
    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    
    # Edit /etc/containerd/config.toml: Find SystemdCgroup = false and set it to true.
    
    # Then restart containerd:
    sudo systemctl restart containerd
    sudo systemctl enable containerd
    
    # Enable kubelet service
    sudo systemctl enable kubelet
    
    # Initialize the cluster (Calico)
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16
    
    # Configure kubectl for your user
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    # Install Calico CNI
    kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/calico.yaml

    kubectl get pods -n kube-system

    # (Optional for lab/demo) Allow scheduling on control-plane
    kubectl taint nodes --all node-role.kubernetes.io/control-plane-

    # k8s get nodes
    kubectl get nodes

# Kubernetes apply for createing pods and svc
    kubectl apply -f triora-deployment.yaml

    kubectl get all -n triora


