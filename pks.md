# Pivotal Container Service(PKS) Hands-on
PKSを利用したK8sクラスタのプロビジョニング、K8sクラスタへのコンテのデプロイを体験します。

## Kubernetesクラスターを作る
``` console
pks login -a https://api.pks.pcflab.jp -k -u <STUDENT_ID> -p <PASSWORD>
```
``` console
pks clusters
Name  Plan Name  UUID  Status  Action

```

``` console
pks plans
Name     ID                                    Description
minimum  8A0E21A8-8072-4D80-B365-D1F502085560  Minimum deployment plan for K8s cluster. Including 1 micro worker and 1 micro master.
small    58375a45-17f7-4291-acf1-455bfdc8e371  Small deployment plan for K8s cluster. Including 3 micro workers and 1 micro master.
medium   241118e5-69b2-4ef9-b47f-4d2ab071aff5  Medium deployment plan for K8s cluster. Including 3 small workers and 1 small master in multi availability zone.
```

```console
pks create-cluster <STUDENT_ID>-k8s --external-hostname=<IP> --plan=minimum

Name:                     kabu-cluster
Plan Name:                minimum
UUID:                     4c54da86-2b2f-404a-9de4-5619493ef74e
Last Action:              CREATE
Last Action State:        in progress
Last Action Description:  Creating cluster
Kubernetes Master Host:   35.189.177.132
Kubernetes Master Port:   8443
Worker Nodes:             1
Kubernetes Master IP(s):  In Progress

Use 'pks cluster kabu-cluster' to monitor the state of your cluster
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
