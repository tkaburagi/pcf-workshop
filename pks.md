# Pivotal Container Service(PKS) Hands-on
PKSを利用したK8sクラスタのプロビジョニング、K8sクラスタへのコンテのデプロイを体験します。
![](https://github.com/tkaburagi/pcf-workshop/blob/master/image/40194879-c876eff0-5a46-11e8-8fae-b8e92383c9f4.png)


## Kubernetesクラスターを作る
``` console
$ pks login -a https://api.pks.pcflab.jp -k -u <STUDENT_ID> -p <PASSWORD>
```
``` console
$ pks clusters
Name  Plan Name  UUID  Status  Action

```

``` console
$ pks plans
Name     ID                                    Description
minimum  8A0E21A8-8072-4D80-B365-D1F502085560  Minimum deployment plan for K8s cluster. Including 1 micro worker and 1 micro master.
small    58375a45-17f7-4291-acf1-455bfdc8e371  Small deployment plan for K8s cluster. Including 3 micro workers and 1 micro master.
medium   241118e5-69b2-4ef9-b47f-4d2ab071aff5  Medium deployment plan for K8s cluster. Including 3 small workers and 1 small master in multi availability zone.
```

```console
$ pks create-cluster <STUDENT_ID>-k8s --external-hostname=<IP> --plan=minimum
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
$ pks clusters
Name             Plan Name  UUID                                  Status       Action
kabu-cluster-10  minimum    4c54da86-2b2f-404a-9de4-5619493ef74e  in progress  CREATE
```
``` console
$ pks cluster <STUDENT_ID>-k8s
Name:                     kabu-cluster
Plan Name:                minimum
UUID:                     4c54da86-2b2f-404a-9de4-5619493ef74e
Last Action:              CREATE
Last Action State:        succeeded
Last Action Description:  Instance provisioning completed
Kubernetes Master Host:   35.189.177.132
Kubernetes Master Port:   8443
Worker Nodes:             1
Kubernetes Master IP(s):  192.168.200.31
```
``` console
$ pks get-credentials <STUDENT_ID>-k8s
Fetching credentials for cluster kabu-cluster-10.
Context set for cluster kabu-cluster-10.

You can now switch between clusters by using:
$kubectl config use-context <cluster-name>
```

## Kubernetesクラスタにログインする
``` console
$ kubectl config use-context <STUDENT_ID>-k8s
Switched to context "kabu-cluster-10".
```

講師にMaster IPを伝える

``` console
$ kubectl cluster-info
Kubernetes master is running at https://35.189.177.132:8443
Heapster is running at https://35.189.177.132:8443/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://35.189.177.132:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
monitoring-influxdb is running at https://35.189.177.132:8443/api/v1/namespaces/kube-system/services/monitoring-influxdb/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
``` console
$ kubectl get all
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.100.200.1   <none>        443/TCP   1h
```
## Kubernetesにアプリをデプロイする
``` console
```
