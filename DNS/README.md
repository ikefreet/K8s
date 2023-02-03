link : https://anggeum.tistory.com/m/entry/Kubernetes-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%84%9C%EB%B9%84%EC%8A%A4Service-Deep-Dive-Cluster-DNS

'''
sudo vim /etc/resolv.conf

nameserver 10.96.0.10
nameserver 8.8.8.8

search openstacklocal
search svc.cluster.local cluster.local
'''
