``` sh
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -v $(which kubectl):/usr/local/mount-from-host/bin/kubectl -v ~/.kube:/.kube -e KUBECONFIG=/.kube/config -t aquasec/kube-bench:latest run -s master

vi /etc/kubernetes/manifests/kube-controller-manager.yaml
# spec:
#   containers:
#   - command:
# ...
#     - --profiling=false
vi /etc/kubernetes/manifests/kube-scheduler.yaml
# spec:
#   containers:
#   - command:
# ...
#     - --profiling=false

docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -v $(which kubectl):/usr/local/mount-from-host/bin/kubectl -v ~/.kube:/.kube -e KUBECONFIG=/.kube/config -t aquasec/kube-bench:latest run -s master
```

``` sh
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -v $(which kubectl):/usr/local/mount-from-host/bin/kubectl -v ~/.kube:/.kube -e KUBECONFIG=/.kube/config -t aquasec/kube-bench:latest run -s node
```
