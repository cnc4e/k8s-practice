``` sh
mkdir /etc/kubernetes/api-server
cat <<EOF > /etc/kubernetes/api-server/audit.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
  - level: Metadata
    resources:
    - group: ""
      resources: ["secrets"]
  - level: Request
    resources:
    - group: ""
      resources: ["pods/log"]
    namespaces: ["test"]
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["pods"]
      resourceNames: ["test"]
    namespaces: ["test"]
  - level: None
EOF

cd /etc/kubernetes/manifests
cp kube-apiserver.yaml kube-apiserver.old
mkdir /var/log/kube-api/
vi kube-apiserver.yaml
...
    - --audit-policy-file=/etc/kubernetes/api-server/audit.yaml
    - --audit-log-path=/var/log/kube-api/audit.log
    - --audit-log-maxage=10
    - --audit-log-maxbackup=3
    - --audit-log-maxsize=10
...
    - mountPath: /etc/kubernetes/api-server
      name: api-server
      readOnly: true
    - mountPath: /var/log/kube-api
      name: kube-api
...
  - hostPath:
      path: /etc/kubernetes/api-server
      type: DirectoryOrCreate
    name: api-server
  - hostPath:
      path: /var/log/kube-api
      type: DirectoryOrCreate
    name: kube-api

kubectl create secret generic test
kubectl create ns test
kubectl run test --image=nginx -n test
kubectl logs -n test test

cat /var/log/kube-api/audit.log | grep Metadata | grep create
cat /var/log/kube-api/audit.log | grep Request
cat /var/log/kube-api/audit.log | grep RequestResponse

kubectl run test --image=nginx
kubectl logs test

cat /var/log/kube-api/audit.log | grep Request
cat /var/log/kube-api/audit.log | grep RequestResponse

kubectl delete secret test
kubectl delete pod test
kubectl delete pod -n test test
kubectl delete ns test

mv kube-apiserver.old kube-apiserver.yaml
```