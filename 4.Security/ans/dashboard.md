``` sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard
# spec:
# ...
#   type: NodePort

kubectl create clusterrolebinding dashboard-view --clusterrole view --serviceaccount kubernetes-dashboard:kubernetes-dashboard

kubectl delete clusterrolebinding dashboard-view 

kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

```