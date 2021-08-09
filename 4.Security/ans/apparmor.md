``` sh
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: busybox
        name: busybox-1
        command: ["sh","-c","sleep 3600"]
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      - image: busybox
        name: busybox-2
        command: ["sh","-c","sleep 3600"]
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: tmp
        hostPath:
          path: /tmp
          type: Directory
EOF

kubectl exec -it <Pod name> -c busybox-1 -- sh
cd /tmp
ls /tmp
touch test-1
ls /tmp
exit

kubectl exec -it <Pod name> -c busybox-2 -- sh
cd /tmp
ls /tmp
touch test-2
ls /tmp
exit

kubectl delete deployment test

# worker
cat /sys/module/apparmor/parameters/enabled
systemctl status apparmor
cat /sys/kernel/security/apparmor/profiles
apparmor_parser -q <<EOF
#include <tunables/global>

profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>

  file,

  # Deny all file writes.
  deny /** w,
}
EOF
cat /sys/kernel/security/apparmor/profiles

# master
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      labels:
        app: test
      annotations:
        container.apparmor.security.beta.kubernetes.io/busybox-1: localhost/k8s-apparmor-example-deny-write
    spec:
      containers:
      - image: busybox
        name: busybox-1
        command: ["sh","-c","sleep 3600"]
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      - image: busybox
        name: busybox-2
        command: ["sh","-c","sleep 3600"]
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: tmp
        hostPath:
          path: /tmp
          type: Directory
EOF

kubectl get pod

kubectl exec -it <Pod name> -c busybox-1 -- sh
cd /tmp
ls /tmp
touch test-3
ls /tmp
exit

kubectl exec -it <Pod name> -c busybox-2 -- sh
cd /tmp
ls /tmp
touch test-4
ls /tmp
rm test-1 test-2 test-4
exit

kubectl delete deployment test

```