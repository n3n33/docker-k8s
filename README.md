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
 sudo repolist
 sudo yum install docker-ce docker-ce-cli containerd.io
```

### run docker 
```
systemctl start docker
```

### kubectl, kubelet, kubeadm  설치
