``` sh
kubectl create secret generic non-encrypted --from-literal=sensitive-data=password

ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
--key=/etc/kubernetes/pki/apiserver-etcd-client.key \
get /registry/secrets/default/non-encrypted
```

``` sh
mkdir /etc/kubernetes/etcd
echo -n "etcd-encrypt-key" | base64
cat <<EOF > /etc/kubernetes/etcd/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: my-key
          secret: ZXRjZC1lbmNyeXB0LWtleQ==
EOF
cp /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/kube-apiserver.yaml
vi /etc/kubernetes/manifests/kube-apiserver.yaml
#spec:
#  containers:
#  - command:
#    - kube-apiserver
#    - --encryption-provider-config=/etc/kubernetes/etcd/encryption-config.yaml # add
#...
#    volumeMounts:
#    - mountPath: /etc/kubernetes/etcd/
#      name: etcd-config
#      readOnly: true
#... 
#  volumes:
#  - hostPath:
#      path: /etc/kubernetes/etcd/
#      type: DirectoryOrCreate
#    name: etcd-config

kubectl create secret generic encrypted --from-literal=sensitive-data=card-number

ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
--key=/etc/kubernetes/pki/apiserver-etcd-client.key \
get /registry/secrets/default/encrypted

ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
--key=/etc/kubernetes/pki/apiserver-etcd-client.key \
get /registry/secrets/default/non-encrypted

kubectl get secrets --all-namespaces -o json | kubectl replace -f -

kubectl get secret encrypted -o yaml
```

``` sh
vi /etc/kubernetes/etcd/encryption-config.yaml
vi /etc/kubernetes/manifests/kube-apiserver.yaml
kubectl get node
vi /etc/kubernetes/manifests/kube-apiserver.yaml

kubectl get secrets --all-namespaces -o json | kubectl replace -f -

ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
--key=/etc/kubernetes/pki/apiserver-etcd-client.key \
get /registry/secrets/default/encrypted

ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
--key=/etc/kubernetes/pki/apiserver-etcd-client.key \
get /registry/secrets/default/non-encrypted

cp /etc/kubernetes/kube-apiserver.yaml /etc/kubernetes/manifests/kube-apiserver.yaml

kubectl delete secret encrypted non-encrypted
```