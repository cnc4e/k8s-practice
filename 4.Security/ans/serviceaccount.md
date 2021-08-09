``` sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: non-mount-token
automountServiceAccountToken: false
EOF

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test
  name: test
spec:
  containers:
  - image: nginx
    name: test
  serviceAccountName: non-mount-token
EOF

kubectl delete pod test
kubectl delete sa non-mount-token
```

``` sh
kubectl create secret generic test --from-literal=test=value -n default
kubectl create secret generic test --from-literal=test=value -n kube-system
kubectl create configmap test --from-literal=test=value -n default
kubectl create configmap test --from-literal=test=value -n kube-system
kubectl create clusterrole get-secret --verb=get --resource=secret
kubectl create clusterrole get-config --verb=get --resource=configmap
kubectl create rolebinding test -n default --clusterrole get-secret --group system:serviceaccounts:default
kubectl create clusterrolebinding test --clusterrole get-config --group system:serviceaccounts

kubectl run test -n default --image=nginx
kubectl exec -it test -- sh
curl -X GET https://kubernetes.default/api/v1/namespaces/default/secrets/test --header "Authorization: Bearer `cat /run/secrets/kubernetes.io/serviceaccount/token`" -k
curl -X GET https://kubernetes.default/api/v1/namespaces/default/configmaps/test --header "Authorization: Bearer `cat /run/secrets/kubernetes.io/serviceaccount/token`" -k

kubectl run test -n kube-system --image=nginx
kubectl exec -it test -n kube-system -- sh
curl -X GET https://kubernetes.default/api/v1/namespaces/kube-system/secrets/test --header "Authorization: Bearer `cat /run/secrets/kubernetes.io/serviceaccount/token`" -k
curl -X GET https://kubernetes.default/api/v1/namespaces/kube-system/configmaps/test --header "Authorization: Bearer `cat /run/secrets/kubernetes.io/serviceaccount/token`" -k

kubectl delete pod test
kubectl delete pod test -n kube-system
kubectl delete clusterrolebinding test
kubectl delete rolebinding test
kubectl delete clusterrole get-secret
kubectl delete clusterrole get-config 
kubectl delete configmap test
kubectl delete configmap test -n kube-system
kubectl delete secret test
kubectl delete secret test -n kube-system
```
