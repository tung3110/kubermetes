1. Cai dat master: 
+ cai dat kubelet, kubeadm, kubectl 
  sudo apt -y install curl apt-transport-https
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
  sudo apt update
  sudo apt -y install vim git curl wget kubelet kubeadm kubectl
  sudo apt-mark hold kubelet kubeadm kubectl

+ tat swap:
  sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
  sudo swapoff -a
+ enable kernel modules and configure sysctl
  # Enable kernel modules
  sudo modprobe overlay
  sudo modprobe br_netfilter

  # Add some settings to sysctl
  sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  net.ipv4.ip_forward = 1
  EOF

  # Reload sysctl
  sudo sysctl --system
+ cai dat docker 
  # Add repo and Install packages
  sudo apt update
  sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  sudo apt update
  sudo apt install -y containerd.io docker-ce docker-ce-cli

  # Create required directories
  sudo mkdir -p /etc/systemd/system/docker.service.d

  # Create daemon json config file
  sudo tee /etc/docker/daemon.json <<EOF
  {
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "100m"
    },
    "storage-driver": "overlay2"
  }
  EOF

  # Start and enable Services
  sudo systemctl daemon-reload 
  sudo systemctl restart docker
  sudo systemctl enable docker
+ Install Mirantis cri-dockerd as Docker Engine shim for Kubernetes
   https://computingforgeeks.com/install-mirantis-cri-dockerd-as-docker-engine-shim-for-kubernetes/
+ sudo systemctl enable kubelet
+ sudo kubeadm config images pull
+ sudo kubeadm config images pull --cri-socket /run/cri-dockerd.sock
+ sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --cri-socket /run/cri-dockerd.sock  \
  --upload-certs
2. Cai dat Worker:
    kubeadm join 192.168.1.38:6443 --token bumb67.arrqnoi1iqe64hxm --discovery-token-ca-cert-hash sha256:5fad6287cb1ff3c766b61b8fbc5830ab1fa02f6f8ff7230de7c9884348fc844d --cri-socket /run/cri-dockerd.sock 
  
