PKS Hands-on

pks login <STUDENT_ID> -p <PASSWORD>
pks clusters
pks plans
pks create-cluster <STUDENT_ID>-k8s --external-hostname=35.194.234.7 --plan=min
pks clusters
pks cluster <STUDENT_ID>-k8s
pks get-credentials <STUDENT_ID>-k8s
kubectl config use-context <STUDENT_ID>-k8s
kubectl cluster-info
kubectl get pods,deployments,services
