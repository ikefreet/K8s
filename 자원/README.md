일반적인 자원 (워크로드)는 kubectl -n mysql-cluster patch pod/mysql-cluster-0 -p '{"metadata":{"finalizers": []}}' --type=merge
<br>
네임스페이스의 경우에는 
kubectl get namespace mysql-cluster -o json \
  | tr -d "\n" | sed "s/\"finalizers\": \[[^]]\+\]/\"finalizers\": []/" \
  | kubectl replace --raw /api/v1/namespaces/"mysql-cluster"/finalize -f -
