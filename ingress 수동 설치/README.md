### Ingress 수동 설치

```
Ingress-nginx 설치

kubectl create -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/baremetal/deploy.yaml

kubectl get all -n ingress-nginx
NAME                                        	READY   STATUS          	RESTARTS   AGE
pod/ingress-nginx-admission-create-z9px5    	0/1 	Completed       	0      	43s
pod/ingress-nginx-admission-patch-9cvbv     	0/1 	Completed       	2      	43s
pod/ingress-nginx-controller-5444f8b96b-6wbtq   0/1 	ContainerCreating   0      	43s

NAME                                     	TYPE    	CLUSTER-IP   	EXTERNAL-IP   PORT(S)                  	AGE
service/ingress-nginx-controller         	NodePort	10.98.92.10  	<none>    	80:31933/TCP,443:30885/TCP   43s
service/ingress-nginx-controller-admission   ClusterIP   10.102.100.252   <none>    	443/TCP                  	43s

NAME                                   	READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   0/1 	1        	0       	43s

NAME                                              	DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-5444f8b96b   1     	1     	0   	43s

NAME                                   	COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1       	21s    	43s
job.batch/ingress-nginx-admission-patch	1/1       	26s    	43s

Ingress nginx Controller를 LoadBalancer Type의 서비스로 운영하도록 설정
kubectl edit services ingress-nginx-controller -n ingress-nginx
=====
…
19  spec:
…
47    type: LoadBalancer
…
=====

kubectl get services -n ingress-nginx
NAME                             	TYPE       	CLUSTER-IP   	EXTERNAL-IP  	PORT(S)                  	AGE
ingress-nginx-controller         	LoadBalancer   10.98.92.10  	192.168.56.200   80:31933/TCP,443:30885/TCP   14m
ingress-nginx-controller-admission   ClusterIP  	10.102.100.252   <none>       	443/TCP                  	14m





—--
Default Ingress Class (Default Ingress Controller) 지정 방법

kubectl get ingressclasses
NAME	CONTROLLER         	PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>   	69m

kubectl edit ingressclasses nginx
=====
…
  7 metadata:
 19   annotations:
 20 	ingressclass.kubernetes.io/is-default-class: "true"
…
=====









```
