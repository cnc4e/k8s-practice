``` sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard
# spec:
# ...
#   type: NodePort

kubectl create clusterrolebinding dashboard-view --clusterrole view --serviceaccount kubernetes-dashboard:kubernetes-dashboard
```

``` sh
kubectl edit deployment -n kubernetes-dashboard kubernetes-dashboard
#    spec:
#      containers:
#      - args:
#        - --namespace=kubernetes-dashboard
#        - --insecure-port=9090
#        - --enable-insecure-login=true
#        - --enable-skip-login=true
# ...
#        ports:
#        - containerPort: 9090

kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard
#   ports:
#   - port: 9090
#     protocol: TCP
#     targetPort: 9090

kubectl delete clusterrolebinding dashboard-view 

kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

```