``` sh
openssl genrsa -out ingress.pem 4096
openssl req -new -key ingress.pem -out ingress.csr
>Common Name (e.g. server FQDN or YOUR name) []:*
openssl x509 -req -days 9999 -in ingress.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ingress.crt

kubectl create secret tls test --cert=ingress.crt --key=ingress.pem

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml

kubectl run nginx --image=nginx
kubectl expose pod nginx --port=80
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test
spec:
  ingressClassName: nginx
  tls:
  - secretName: test
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
EOF

kubectl delete ingress test
kubectl delete secret test
kubectl delete pod nginx
kubectl delete svc nginx
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml
```