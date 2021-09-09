``` sh
# master
cd /etc/kubernetes/manifests
cp kube-apiserver.yaml kube-apiserver.old
vi kube-apiserver.yaml
>     - --enable-admission-plugins=NodeRestriction,PodSecurityPolicy

cat <<EOF | kubectl apply -f -
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: test
spec:
  privileged: false
  volumes:
  - 'emptyDir'
  hostNetwork: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
  readOnlyRootFilesystem: true
EOF

cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: test
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames: ["test"]
EOF

kubectl create ns test

cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: test
  namespace: test
roleRef:
  kind: ClusterRole
  name: test
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  apiGroup: rbac.authorization.k8s.io
  name: system:serviceaccounts:test
EOF

kubectl run nginx --image=nginx -n test
kubectl describe pod nginx -n test
kubectl delete pod nginx -n test

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: test
spec:
  containers:
  - image: nginx
    name: nginx
    securityContext:
      privileged: false            # これはなくてもPSPで自動で設定される
      readOnlyRootFilesystem: true # これはなくてもPSPで自動で設定される
      runAsUser: 101
    volumeMounts:
    - mountPath: /var/run/
      name: var-run
    - mountPath: /var/cache/nginx
      name: var-cache-nginx
  volumes:
  - name: var-run
    emptyDir: {}
  - name: var-cache-nginx
    emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF

kubectl get pod -n test

kubectl delete pod nginx -n test
kubectl delete rolebindig test -n test
kubectl delete clusterrole test
kubectl delete ns test
kubectl delete psp test

vi kube-apiserver.yaml
>     - --enable-admission-plugins=NodeRestriction

```