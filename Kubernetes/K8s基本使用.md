

## Deployment创建、删除

```shell
kubectl create -f test.yaml


kubectl delete -f test.yaml



```

## Service创建和查看

```shell
kubectl create -f test.yaml

kubectl get service --all-namespaces
```



## Pod查看

```shell
# 查看pod
kubectl get pod --all-namespaces -o wide

# pod详细信息
kubectl describe pod pod-name -n namespace-name

# 删除pod
kubectl delete pod pod-name -n namespace-name
```

