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
  
qA> Master:
1. Ch???y script install-k8s.sh (nh???n ph??m 'q' ????? ch???y ti???p n???u ???????c h???i)
	- N???u b??o l???i "unknown service runtime.v1alpha2.ImageService" th?? l??m nh?? sau:		
	Comment disabled_plugins = ["cri"] if exist in /etc/containerd/config.toml, then running systemctl restart containerd worked for me.	
2. Ch???y l???nh sau tr??n node master:
	sudo kubeadm init --apiserver-advertise-address=$(hostname -i) --pod-network-cidr=192.168.0.0/16 
	sudo kubeadm init --apiserver-advertise-address=10.0.11.201 --pod-network-cidr=192.168.0.0/16 
3. Ch???y ti???p c??c l???nh nh?? terminal h?????ng d???n:
		mkdir -p $HOME/.kube
		sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		sudo chown $(id -u):$(id -g) $HOME/.kube/config

	-> Ch???y th??m l???nh sau n???u l???nh kubectl cluster-info b??o l???i: sudo chown -R $USER  /etc/kubernetes/admin.conf
	
4. C??i ?????t m???ng
	4.1 D??ng l???nh: kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
	4.2 Enable c???ng: 6443
5. Th??? 1 deployment.
6. Get token ????? c??c node join v??o m???ng:
	kubeadm token create --print-join-command
7. N???u c??i tr??n EC2, th?? b??? sung c??c thao t??c sau:
	- Allow custom protocol, d???ng "Ip-in-Ip (4)" trong security group, https://stackoverflow.com/questions/60806708/kubernetes-with-calico-on-aws-cannot-ping-pods-on-on-different-nodes
	- Disable source/destination checks: https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html#EIP_Disable_SrcDestCheck
	
B> Worker:
1. Ch???y script install-k8s.sh
2. Join v??o m???ng master d??ng token thu ???????c ph??a master: sudo kubeadm join ....

C> C??i ?????t kh??c
	1. C??i ?????t docker credential:
		- Login: docker login
		- Run: kubectl create secret generic regcred --from-file=.dockerconfigjson=/home/ubuntu/.docker/config.json --type=kubernetes.io/dockerconfigjson
	
	Ho???c: kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
	2. C??i ?????t kong-ingress controller: L???y file kong-ingress-controller-install.yaml, ???? custom ????? expose kong-proxy ra NodePort 32000
	3. Dashboard:
		- T???o secret: kubectl create secret tls stag-pccc.bsafe.vn --cert=k8s/stag-pccc.bsafe.vn.fullchain.pem --key=k8s/stag-pccc.bsafe.vn.privkey.pem -n kubernetes-dashboard
		- L???y token dashboard: kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')



