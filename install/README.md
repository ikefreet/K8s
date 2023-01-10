### Install with Kubespray
```
Kubespray를 사용하여 Kubernetes Cluster 구성

(kube-control1에서)

cat /etc/fstab

- - -
SSH Key Pair 생성 및 공개키 배포 (키 기반 인증 설정)

ssh-keygen
ssh-copy-id vagrant@localhost
ssh-copy-id vagrant@192.168.56.21
ssh-copy-id vagrant@192.168.56.22
ssh-copy-id vagrant@192.168.56.23

- - -

Kubespray로 Kubernetes Cluster 배포 설정

cd ~
git clone -b v2.18.1 https://github.com/kubernetes-sigs/kubespray 
cd ~/kubespray

sudo apt-get update
sudo apt-get install python3 python3-pip -y

sudo pip3 install -r requirements.txt

cp -rfp inventory/sample/ inventory/mycluster
vi inventory/mycluster/inventory.ini
===
[all]
kube-control1    ansible_host=192.168.56.11    ip=192.168.56.11    ansible_connection=local
kube-node1    ansible_host=192.168.56.21    ip=192.168.56.21
kube-node2    ansible_host=192.168.56.22    ip=192.168.56.22
kube-node3    ansible_host=192.168.56.23    ip=192.168.56.23

[all:vars]
ansible_python_interpreter=/usr/bin/python3

[kube_control_plane]
kube-control1

[etcd]
kube-control1

[kube_node]
kube-node1
kube-node2
kube-node3

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
===


vi inventory/mycluster/group_vars/k8s_cluster/addons.yml
===
  16  metrics_server_enabled: true
  93  ingress_nginx_enabled: true
139  metallb_enabled: true
141  metallb_ip_range:
142    - "192.168.56.200-192.168.56.210"
168  metallb_protocol: "layer2"
===

vi inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml
===
129  kube_proxy_strict_arp: true
===


Kubernetes Cluster 배포

ansible all -i inventory/mycluster/inventory.ini -m ping
ansible all -i inventory/mycluster/inventory.ini -m apt -a "update_cache=yes" --become

ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml --become

mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown vagrant:vagrant ~/.kube/config

kubectl get nodes
NAME        	STATUS   ROLES              	AGE   VERSION
kube-control1   Ready	control-plane,master   24m   v1.22.8
kube-node1  	Ready	<none>             	23m   v1.22.8
kube-node2  	Ready	<none>             	23m   v1.22.8
kube-node3  	Ready	<none>             	23m   v1.22.8
```

<br>


### K8s 수동 설치
```
kubeadm을 이용한 Kubernetes Cluster 수동 구성

(모든 노드에서 진행)

sudo swapoff -a
sudo vim /etc/fstab

APT Repository 정보 갱신
sudo apt-get update -y

추가 Repository 정보 등록을 위한 패키지 설치
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release -y

Docker 설치를 위한 추가 Repository 정보 등록
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io -y

sudo docker info | grep -i cgroup
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
WARNING: No swap limit support

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

sudo systemctl daemon-reload
sudo systemctl restart docker.service

sudo docker info | grep -i cgroup
WARNING: No swap limit support
 Cgroup Driver: systemd
 Cgroup Version: 1

Kubernetes 설치를 위한 추가 Repository 정보 등록
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y


sudo apt-get install -y kubelet=1.22.10-00 kubeadm=1.22.10-00 kubectl=1.22.10-00

sudo apt-mark hold kubelet kubeadm kubectl

------------

– (kube-control1에서만) –

Kubernetes Cluster 구성
sudo kubeadm init --control-plane-endpoint 192.168.56.11 --pod-network-cidr 192.168.0.0/16 --apiserver-advertise-address 192.168.56.11


Kubernetes Cluster 관리를 위한 관리자 인증정보 등록
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Kubernetes Cluster 네트워크 구축을 위한 Calico CNI 설치
kubectl create -f https://docs.projectcalico.org/manifests/calico.yaml

kubectl get nodes
NAME            STATUS     ROLES    AGE   VERSION
kube-control1   NotReady   master   54s   v1.22.10
----

-- (kube-node1 ~ kube-node3에서만) ---
sudo kubeadm join 192.168.56.11:6443 --token (k8s token) \
 --discovery-token-ca-cert-hash (k8s discovery-token-ca-cert-hash)

-----

– (kube-control1에서만) –
vagrant@kube-control1:~$ kubectl get nodes
NAME        	STATUS 	ROLES              	AGE   VERSION
kube-control1   Ready  	control-plane,master   14m   v1.22.10
kube-node1  	NotReady   <none>             	56s   v1.22.10
kube-node2  	NotReady   <none>             	46s   v1.22.10
kube-node3  	NotReady   <none>             	41s   v1.22.10

vagrant@kube-control1:~$ kubectl get nodes
NAME        	STATUS   ROLES              	AGE   VERSION
kube-control1   Ready	control-plane,master   27m   v1.22.10
kube-node1  	Ready	<none>             	13m   v1.22.10
kube-node2  	Ready	<none>             	13m   v1.22.10
kube-node3  	Ready	<none>             	13m   v1.22.10

----


```
