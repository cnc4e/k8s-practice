``` sh
# worker
mkdir /var/lib/kubelet/seccomp
cat <<EOF > /var/lib/kubelet/seccomp/all-error.json
{
    "defaultAction": "SCMP_ACT_ERRNO"
}
EOF

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
    seccompProfile: 
      type: Localhost
      localhostProfile: all-error.json
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
EOF
kubectl apply -f pod.yaml
kubectl describe pod nginx
kubectl delete -f pod.yaml

# worker
wget https://downloads.dockerslim.com/releases/1.36.2/dist_linux.tar.gz
tar -xvf dist_linux.tar.gz
mv dist_linux/docker-slim /usr/local/bin/
mv dist_linux/docker-slim-sensor /usr/local/bin/
docker-slim profile nginx
docker-slim build nginx
ll /tmp/docker-slim-state/.docker-slim-state/images/08b152afcfae220e9709f00767054b824361c742ea03a9fe936271ba520a0a4b/artifacts
mv /tmp/docker-slim-state/.docker-slim-state/images/08b152afcfae220e9709f00767054b824361c742ea03a9fe936271ba520a0a4b/artifacts/nginx-seccomp.json /var/lib/kubelet/seccomp/
vi /var/lib/kubelet/seccomp/nginx-seccomp.json
> add fstatfs

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
    seccompProfile: 
      type: Localhost
      localhostProfile: nginx-seccomp.json
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
EOF
kubectl apply -f pod.yaml
kubectl describe pod nginx
kubectl delete -f pod.yaml
```