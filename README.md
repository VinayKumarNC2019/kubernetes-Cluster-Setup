# kubernetes-Cluster-Setup
To setup kubernetes cluster on centos 7

# Assumptions:

* CPU 2 cores
* RAM 2GB 
* CentOs 7
* Sudo user kube required, but all commands executed as root, except few commands which are executed after kubeadm init


# Disable SELINUX

sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

setenforce 0

# Disable firewalld

systemctl disable --now firewalld

# Disable Swap

swapoff -a

vim /etc/fstab  #comment Swap line

# Install Docker CE

yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

yum install -y docker-ce

# Install Kubernetes

# Add repo for kubernetes components like kubeadm,kubelet,kubectl

vi /etc/yum.repos.d/kubernetes.repo

    
[kubernetes]

name=Kubernetes

baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64

enabled=1

gpgcheck=1

repo_gpgcheck=1

gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
        
--------------------------------------------------

yum install -y kubelet kubeadm kubectl
 
# Reboot the server

sudo reboot

# Enable and start docker and kubelet (kubelet may fail to start, ignore for now)

systemctl start docker && systemctl enable docker

systemctl start kubelet && systemctl enable kubelet

# Change the cgroup-driver (We need to make sure the docker-ce and kubernetes are using same 'cgroup'.)

docker info | grep -i cgroup   #And you see the docker is using 'cgroupfs' as a cgroup-driver.

sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf  

# Reload the systemd system and restart the kubelet service.

systemctl daemon-reload

systemctl restart kubelet

# Kubernetes Cluster Initialization

kubeadm init --apiserver-advertise-address=10.128.0.10 --pod-network-cidr=10.128.0.0/16  # {master ip:  10.128.0.10  pod network :                                                                                                   10.128.0.0/16 }

Note:

-- apiserver-advertise-address = determines which IP address Kubernetes should advertise its API server on.

-- pod-network-cidr = specify the range of IP addresses for the pod network. 
   We're using the 'flannel' virtual network. If you want to use another pod network such as weave-net or calico, change the range IP      address.
   
# Below message displayed 

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.128.0.10:6443 --token oagfbq.abnjmjojco1v42q1 \
    --discovery-token-ca-cert-hash sha256:3afc010c09a40d7e2885dc524a298217d250390ad454c045c60ff7815fec933c

  
# Switce to normal Sudo user kube and execute below commands

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

# To deploy flannel network

kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml

------------------------------------------------------------------------------------------

In case if you would like to destroy and rebuild the cluster: 

On Master and worker Node:

sudo kubeadm reset 

rm -f  /etc/cni/net.d

sudo rm -rf /var/lib/cni/

systemctl daemon-reload

systemctl restart kubelet

sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X


