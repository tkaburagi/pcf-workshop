# PKS Hands-on

## Kubernetesクラスターを作る
pks login <STUDENT_ID> -p <PASSWORD>

pks clusters

pks plans

pks create-cluster <STUDENT_ID>-k8s --external-hostname=<IP> --plan=min

pks clusters

pks cluster <STUDENT_ID>-k8s

pks get-credentials <STUDENT_ID>-k8s

## Kubernetesクラスタにログインする
kubectl config use-context <STUDENT_ID>-k8s

kubectl cluster-info

kubectl get pods,deployments,services

## Kubernetesにアプリをデプロイする
