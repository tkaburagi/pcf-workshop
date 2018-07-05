# Pivotal Container Service(PKS) Hands-on
PKSを利用したK8sクラスタのプロビジョニング、K8sクラスタへのコンテのデプロイを体験します。

## Kubernetesクラスターを作る
``` console
pks login <STUDENT_ID> -p <PASSWORD>
```
``` console
pks clusters
```

``` console
pks plans
pks create-cluster <STUDENT_ID>-k8s --external-hostname=<IP> --plan=min
```

``` console
pks clusters
pks cluster <STUDENT_ID>-k8s
pks get-credentials <STUDENT_ID>-k8s
```

## Kubernetesクラスタにログインする
``` console
kubectl config use-context <STUDENT_ID>-k8s
```

``` console
kubectl cluster-info
kubectl get pods,deployments,services
```
## Kubernetesにアプリをデプロイする
