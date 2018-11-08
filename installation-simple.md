# Kubernetes installation
Kubernetes installation in a simple way.

Kubernetes has to be seen as a Cluster of Worker Nodes and a Cluster of Master Nodes. Master Nodes can be reduced to 1 in if High-Availibility is not mandatory about Kubernetes API Resources. Worker Nodes has to be sufficient in system resources way (CPU, RAM and Disk) and clustered by 2 nodes minimum for balancing and availibility.

A multiple way to install Kubernetes are available. We will focus on Kubeadm way to explain a few things. We need to know that other ways exist like Kube-Spray that will spread a entire Cluster through Ansible roles and playbooks.

*Our way will be Kubeadm*

## Installation
This Installation part is for both Master and Workers Nodes.

### Pre-requisites
System has to be changed from usual behaviour in order to be able to accept Kubernetes infrastructure.
To preserve us from all the fashioned  about SELinux it will be disabled.
A big thing (and not usual) is to absolutely disable Swap not to stuck containers on disk memory... that is a bad thing.
About kernel we have to enable ipv6 bridge and ipv4 forward to allow docker network to contact physical/virtual network connection.

`root@master, root@worker`
```
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
swapoff `cat /etc/fstab | grep swap | awk '{print $1}'`
sed -i 's/^\([^#].*swap.*\)$/#\1/g' /etc/fstab
mount -a
sed -i 's/^\(net.ipv4.ip_forward\).*/\1 = 1/g' /etc/sysctl.conf
sysctl -p
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### Install Docker CE
Kubernetes is 100% compatible with Docker CE 17.03.
This contraints us to allow obsolete packages on CentOS/RedHat 7 servers.
Once done we don't forget to start and enable service on boot.

`root@master, root@worker`
```
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y --setopt=obsoletes=0 \
  docker-ce-17.03.2.ce-1.el7.centos \
  docker-ce-selinux-17.03.2.ce-1.el7.centos
systemctl enable docker
systemctl start docker
systemctl status docker
```

Docker status has to be good to continue on next section.

### Install Kubernetes
Kubernetes top stability is eligible with 1.9.7. We can perfectly install much recent version without any trouble. We will just install this stable version to prevent everything aside.

`root@master, root@worker`
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet-1.9.7 kubeadm-1.9.7 kubectl-1.9.7
```

With Kubernetes version 1.9.7 we need to set a little configuration to point on good unix cgroup libraries.

```
sed -i 's/^\(Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=\).*/\1cgroupfs"/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

Let's start the monster !
```
systemctl daemon-reload
systemctl enable kubelet
systemctl restart kubelet
systemctl status kubelet
```

Kubernetes status immediately after (re)starting should be "active" and running.
Don't panic if it fails when retrying to show status. In fact K8s is repetitively looking for a Global Network  that will come in next steps. As it is not the cas yet, status reveals failing status.


## Master Configuration

### Network initialisation

Master node host the Global Network configuration. We need to instanciate it.

`root@master`
```
kubeadm init --pod-network-cidr 10.244.0.0/16
```

After some system event (reboot or kubelet/kubeadm re-install) kubeadm init may crash with a big ERROR CRI. In this case you have to delete the directory /etc/cni. If not possible you can add the flag --ignore-preflight-errors=cri to bypass this error : kubeadm init --pod-network-cidr 10.244.0.0/16 --ignore-preflight-errors=cri

`root@master`
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

### Network Configuration
With Kubernetes everything is module even networking. We need to choice our Network Configuration between a lot. We expose 2 choices here.

#### Choice 1: Flannel
Flannel driver is one of the basics one to easily use Kubernetes on the run. It may be coupled with Calico for Network Policies.
`root@master`
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### Choice 2: Calico
`root@master`
```
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/etcd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/rbac.yaml
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/calico.yaml
```

Here we go ! You Cluster is almost ready !

## Worker Nodes
It just left to add Worker Nodes in your Cluster to be Ready.

At the end of the Network Configuration Kubernetes shows a command to play. If you don't get it don't worry  will show up again ;)

`root@master`
```
kubeadm token create --print-join-command
```
Now paste it on worker nodes.

### Join Worker Nodes to the Cluster
Paste the Cluster join-command on every Worker Nodes to add them in the Kubernetes Cluster. It should look like :
```
kubeadm join --token b26bb6.e040d60287e35dc7 192.168.248.166:6443 --discovery-token-ca-cert-hash sha256:0a7dfc557cc247b630ec20e58e3f367f53c27b1411d491542a9395ba2dc48e3a
```

## Cluster Status
Now ensure you nodes has been well added to the Cluster.

`root@master`
```
kubectl get nodes
```
Output:
```
NAME         STATUS    ROLES     AGE       VERSION
evx4406969   Ready     <none>    5h        v1.9.7
evx4406970   Ready     master    5h        v1.9.7
```

Everything is Ready!


## Troubleshooting

This is a quick list of regular error we can encounter.

### CRI Error

You can get a CRI ERROR on Master when initializing global network and Worker Nodes when joining the cluster.
Usually you need to uninstall Kubernetes (and Docker) then delete the directory /etc/cni to install again.
Another option is to add --ignore-preflight-errors=cri to commands failing.
This happen when uninstall is not properly done.

### Resources limit
Another hint to follow is "Do we have enough resources to keep going ?"
Check your CPU and Memory. You might be faced to not enough RAM to properly run all basics Pods on Master Node.
A Master Node with 4Go RAM was not enough to clearly run it. An upgrade to 6Go solve the problem.

A encountered case was the Pods "kubedns" was not able to start and showed up a "FailedCreatePodSandBox".
