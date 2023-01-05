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



### K8s 수동 설
