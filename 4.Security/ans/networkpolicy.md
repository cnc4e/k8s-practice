``` sh
kubectl run nginx --image nginx
kubectl exec nginx -- curl -s http://169.254.169.254/latest/meta-data/

cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: metadata-deny
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32
EOF

kubectl exec nginx -- curl -s http://169.254.169.254/latest/meta-data/
kubectl exec nginx -- curl -s 142.251.32.14 # google.com

kubectl delete pod nginx
kubectl delete network policy metadata-deny
```

``` sh
kubectl create ns public
kubectl create ns private

cat <<EOF | kubectl apply -f -
---
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
---
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
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: front
  name: front
  namespace: public
spec:
  replicas: 1
  selector:
    matchLabels:
      app: front
  template:
    metadata:
      labels:
        app: front
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
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: back
  name: back
  namespace: public
spec:
  replicas: 1
  selector:
    matchLabels:
      app: back
  template:
    metadata:
      labels:
        app: back
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
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: front
  name: front
  namespace: private
spec:
  replicas: 1
  selector:
    matchLabels:
      app: front
  template:
    metadata:
      labels:
        app: front
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
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: back
  name: back
  namespace: private
spec:
  replicas: 1
  selector:
    matchLabels:
      app: back
  template:
    metadata:
      labels:
        app: back
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
---
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
---
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
---
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
---
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
```