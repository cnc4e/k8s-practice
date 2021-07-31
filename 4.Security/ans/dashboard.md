``` sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

kubectl edit deployment -n kubernetes-dashboard kubernetes-dashboard
kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard
```