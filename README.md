# docker-k8s 환경 구축해보기
 <img src="https://img.shields.io/badge/-Docker-2496ED?style=flat&logo=Docker&logoColor=white"/> <img src="https://img.shields.io/badge/-Kubernetes-326CE5?style=flat&logo=Kubernetes&logoColor=white"/>

1. k8s master,worker1, worker2 -> 구성

### docker engine 설치
```
vi /etc/yum.repos.d/docker-ce.repo
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://download.docker.com/linux/centos/$releasever/source/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://download.docker.com/linux/centos/$releasever/source/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://download.docker.com/linux/centos/$releasever/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg
```
```
 sudo yum install -y yum-utils
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
 sudo yum clean all
 sudo yum repolist
 sudo yum install docker-ce docker-ce-cli containerd.io
```

### run docker 
```
systemctl start docker
```

### kubectl, kubelet, kubeadm  설치

```
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker

sudo systemctl restart kubelet
```

CPU 2개 이하일때
```
kubeadm init --pod-network-cidr=20.96.0.0/12 --apiserver-advertise-address=192.168.56.103 --v=5 --ignore-preflight-errors=NumCPU
```

실행 성공 후 로그
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join masterIP:6443 --token 3g0m9o.vso9b2fmwytlzi88 \
        --discovery-token-ca-cert-hash sha256:d4ffcc6202efc6c738f80d41fa1734a1d12b813b693d4f151a98b2147bf71ac2 --v=5 \
        --ignore-preflight-errors=NumCPU
```

```
# kubectl get nodes
NAME        STATUS     ROLES                  AGE    VERSION
test1.com   NotReady   control-plane,master   111m   v1.23.5
test2.com   NotReady   <none>                 60s    v1.23.5
test3.com   NotReady   <none>                 57s    v1.23.5
```

## 네트워크 실행
```
wget https://docs.projectcalico.org/manifests/calico.yaml
sed -i -e 's?192.168.0.0/16?192.168.10.0/24?g' calico.yaml
kubectl apply -f calico.yaml
kubectl get pods --namespace kube-system
```

## k8s reset
```
# 각 노드별로 해야됨
kubeadm reset
systemctl restart docker
systemctl restart kubelet
```
