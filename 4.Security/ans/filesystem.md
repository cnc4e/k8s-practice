``` sh
cat <<EOF > pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    securityContext:
      readOnlyRootFilesystem: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF

kubectl apply -f pod.yaml
kubectl logs nginx
kubectl delete -f pod.yaml

cat <<EOF > pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    securityContext:
      readOnlyRootFilesystem: true
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

kubectl apply -f pod.yaml
kubectl logs nginx
kubectl delete -f pod.yaml
```