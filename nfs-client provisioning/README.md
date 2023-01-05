### NFS-Client provisioning

```
<PV 동적 볼륨 프로비저닝을 위한 Provisioner 구성>
NFS-subdir-external-provisioner
https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

NFS 서버 구성
—
vagrant@kube-control1:~/work/20221125$ sudo -i
root@kube-control1:~# vim /etc/exports
root@kube-control1:~# cat /etc/exports
/nfsvolume *(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs-volume *(rw,sync,no_subtree_check,no_root_squash)
root@kube-control1:~#
root@kube-control1:~# mkdir /nfsvolume
root@kube-control1:~# exportfs -arv
exporting *:/srv/nfs-volume
exporting *:/nfsvolume
—

git clone https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

cd nfs-subdir-external-provisioner/
kubectl create -f rbac.yaml
vim deployment.yaml
=====
…
spec:
...
  template:
...
	spec:
...
      	env:
        	- name: PROVISIONER_NAME
          	value: k8s-sigs.io/nfs-subdir-external-provisioner
        	- name: NFS_SERVER
          	value: (Control Plane IP)
        	- name: NFS_PATH
          	value: /nfsvolume
  	volumes:
    	- name: nfs-client-root
      	nfs:
        	server: (Control Plane IP)s
        	path: /nfsvolume
…
=====

kubectl create -f deployment.yaml
kubectl create -f class.yaml

—--


```
