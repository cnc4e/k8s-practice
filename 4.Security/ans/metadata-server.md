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
kubectl delete networkpolicy metadata-deny
```