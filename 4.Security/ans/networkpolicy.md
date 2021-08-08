``` sh
kubectl create ns public
kubectl create ns private

cat <<EOF > public-default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: public
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
EOF
cat <<EOF > private-default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: private
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
EOF
cat <<EOF > public-front.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: front
  name: front
  namespace: public
spec:
  containers:
  - image: nginx
    name: nginx
    startupProbe:
      exec:
        command:
        - sh
        - -c
        - "echo public-front > /usr/share/nginx/html/index.html"
EOF
cat <<EOF > public-back.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: back
  name: back
  namespace: public
spec:
  containers:
  - image: nginx
    name: nginx
    startupProbe:
      exec:
        command:
        - sh
        - -c
        - "echo public-back > /usr/share/nginx/html/index.html"
EOF
cat <<EOF > private-front.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: front
  name: front
  namespace: private
spec:
  containers:
  - image: nginx
    name: nginx
    startupProbe:
      exec:
        command:
        - sh
        - -c
        - "echo private-front > /usr/share/nginx/html/index.html"
EOF
cat <<EOF > private-back.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: back
  name: back
  namespace: private
spec:
  containers:
  - image: nginx
    name: nginx
    startupProbe:
      exec:
        command:
        - sh
        - -c
        - "echo private-back > /usr/share/nginx/html/index.html"
EOF
cat <<EOF > public-front-svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: front
  name: front
  namespace: public
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: front
EOF
cat <<EOF > public-back-svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: back
  name: back
  namespace: public
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: back
EOF
cat <<EOF > private-front-svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: front
  name: front
  namespace: private
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: front
EOF
cat <<EOF > private-back-svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: back
  name: back
  namespace: private
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: back
EOF
cat <<EOF > public-allow-front-to-back.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-front-to-back
  namespace: public
spec:
  podSelector:
    matchLabels:
      app: front
  policyTypes:
  - Egress
  egress:
    - to:
      - podSelector:
          matchLabels:
            app: back
EOF
cat <<EOF > public-allow-back-from-front.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-back-from-front
  namespace: public
spec:
  podSelector:
    matchLabels:
      app: back
  policyTypes:
  - Ingress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            app: front
EOF
cat <<EOF > private-allow-front-to-back.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-front-to-back
  namespace: private
spec:
  podSelector:
    matchLabels:
      app: front
  policyTypes:
  - Egress
  egress:
    - to:
      - podSelector:
          matchLabels:
            app: back
EOF
cat <<EOF > private-allow-back-from-front.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-back-from-front
  namespace: private
spec:
  podSelector:
    matchLabels:
      app: back
  policyTypes:
  - Ingress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            app: front
EOF
cat <<EOF > allow-public-front-to-private-back.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-public-front-to-private-back
  namespace: public
spec:
  podSelector:
    matchLabels:
      app: front
  policyTypes:
  - Egress
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: private
      - podSelector:
          matchLabels:
            app: front
EOF
cat <<EOF > allow-private-front-from-public-front.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-private-front-from-public-front
  namespace: private
spec:
  podSelector:
    matchLabels:
      app: front
  policyTypes:
  - Ingress
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: public
      - podSelector:
          matchLabels:
            app: front
EOF
cat <<EOF > private-to-public.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: private-to-public
  namespace: private
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: public
EOF
cat <<EOF > public-from-private.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: public-from-private
  namespace: public
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: private
EOF

kubectl apply -f .

kubectl exec -n public front -- curl -s -m 1 back.public 2>/dev/null
kubectl exec -n public front -- curl -s -m 1 front.private 2>/dev/null
kubectl exec -n public front -- curl -s -m 1 back.private 2>/dev/null
kubectl exec -n public front -- curl -s -m 1 142.250.191.142 2>/dev/null

kubectl exec -n public back -- curl -s -m 1 front.public 2>/dev/null
kubectl exec -n public back -- curl -s -m 1 front.private 2>/dev/null
kubectl exec -n public back -- curl -s -m 1 back.private 2>/dev/null
kubectl exec -n public back -- curl -s -m 1 142.250.191.142 2>/dev/null

kubectl exec -n private front -- curl -s -m 1 front.public 2>/dev/null
kubectl exec -n private front -- curl -s -m 1 back.public 2>/dev/null
kubectl exec -n private front -- curl -s -m 1 back.private 2>/dev/null
kubectl exec -n private front -- curl -s -m 1 142.250.191.142 2>/dev/null

kubectl exec -n private back -- curl -s -m 1 front.public 2>/dev/null
kubectl exec -n private back -- curl -s -m 1 back.public 2>/dev/null
kubectl exec -n private back -- curl -s -m 1 front.private 2>/dev/null
kubectl exec -n private back -- curl -s -m 1 142.250.191.142 2>/dev/null

kubectl delete -f .
kubectl delete ns public private

```