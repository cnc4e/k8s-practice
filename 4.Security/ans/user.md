``` sh
kubectl create clusterrolebinding add-user --clusterrole=view --user=add-user

cd /etc/kubernetes/pki
openssl genrsa -out user-add-test.pem 4096
openssl req -new -key user-add-test.pem -out user-add-test-csr.pem
...
Common Name (e.g. server FQDN or YOUR name) []:add-user
...

openssl x509 -req -days 9999 -in user-add-test-csr.pem -CA ca.crt -CAkey ca.key -CAcreateserial -out user-add-test-crt.pem

cat user-add-test-crt.pem | base64
user-add-test.pem | base64
cp ~/.kube/config ~/.kube/config-add-user
vi ~/.kube/config-add-user
> ユーザ名をadd-userに修正
> client-certificate-dataとclient-key-dataを先ほどbase64エンコードしたcrtとkeyに修正

export KUBECONFIG=~/.kube/config-add-user
kubectl get pod -A
kubectl run test --image=nginx

export KUBECONFIG=~/.kube/config
kubectl create role allow-create-secret --resource secret --verb create,get,list
kubectl create rolebinding add-user-create-secret --role=allow-create-secret --user=add-user

export KUBECONFIG=~/.kube/config-add-user
kubectl get secret
kubectl create secret generic add-user --from-literal=test-key=test-value

export KUBECONFIG=~/.kube/config
kubectl delete secret add-user
kubectl delete rolebinding add-user-create-secret
kubectl delete role allow-create-secret
kubectl delete clusterrolebinding add-user 
```

