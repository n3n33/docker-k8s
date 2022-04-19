# docker-k8s 환경 구축해보기
 <img src="https://img.shields.io/badge/-Docker-2496ED?style=flat&logo=Docker&logoColor=white"/> <img src="https://img.shields.io/badge/-Kubernetes-326CE5?style=flat&logo=Kubernetes&logoColor=white"/>

1. k8s master,worker1, worker2 -> 구성

### docker engine 설치
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
vi /etc/yum.reposd/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl

yum clean all
yum repolist
yum install kubelet kubeadm kubectl -y
```

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

## 네트워크 플러그인
```
wget https://docs.projectcalico.org/manifests/calico.yaml
sed -i -e 's?192.168.0.0/16?192.168.10.0/24?g' calico.yaml
kubectl apply -f calico.yaml
kubectl get pods --namespace kube-system
```
## k8s worker join
```
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


## k8s log 확인
```
kubectl logs -n kube-system calico-node-67rs7
```

## k8s reset
```
# 각 노드별로 해야됨
kubeadm reset
systemctl restart docker
systemctl restart kubelet
```

## dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
kubectl -n kubernetes-dashboard edit service kubernetes-dashboard
## type: ClusterIP -> type: NodePort 변경
kubectl -n kubernetes-dashboard get service kubernetes-dashboard 

## serviceaccount 생성

cat <<EOF | kubectl create -f -
 apiVersion: v1
 kind: ServiceAccount
 metadata:
   name: admin-user
   namespace: kube-system
EOF

## ClusterRoleBinding 생성

cat <<EOF | kubectl create -f -
 apiVersion: rbac.authorization.k8s.io/v1
 kind: ClusterRoleBinding
 metadata:
   name: admin-user
 roleRef:
   apiGroup: rbac.authorization.k8s.io
   kind: ClusterRole
   name: cluster-admin
 subjects:
 - kind: ServiceAccount
   name: admin-user
   namespace: kube-system
EOF

## token 복사
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}') 
```
![k8s-cap](https://user-images.githubusercontent.com/51447153/163783411-f8ed1503-65dd-463e-85b3-32c8f14a59d6.png)
