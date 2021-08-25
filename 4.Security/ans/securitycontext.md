``` sh
# master
cat <<EOF > pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  securityContext:
    fsGroup: 101
  containers:
  - image: nginx
    name: nginx
    securityContext:
      privileged: false
      readOnlyRootFilesystem: true
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

kubectl apply -f pod.yaml
kubectl exec -it pod -- sh
id
ls -ld /var/run/
ls -ld /var/cache/nginx
exit

# worker
ps -ef | grep ngixn
id 101

# master
kubectl delete -f pod.yaml
```