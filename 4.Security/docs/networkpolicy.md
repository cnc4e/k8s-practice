[トップ](../README.md)  
前： [ingressでTLSの終端](ingress-tls.md)  

---

# NetworkPolicyによるアクセス制御

ネットワークポリシーはK8s内のSecurityGroupのようなものです。

1. 以下を満たリソース群をデプロイしてください。

- Namespaceを2つ。名前は`public`と`private`
- 各Namespaceにはすべてのインバウンドとアウトバウンドを禁止するNetworkPolicyがついている。ただし、CoreDNSと通信するためTCP/UDP53のアウトバウンドは許可する
- 各Namespaceには2つのPodがある。すべてnginxを動かす。名前は`front`と`back`でそれぞれ`app=<Pod名>`のラベルがついている
- 各Podの/usr/share/nginx/html/index.htmlはどのNamespaceのどのDeploymentか判別できる内容にする。(例：public-fornt、private-back等)
- 各PodはPod名と同じ名前のService経由でport:80にアクセスできる。
- 各Namespace内で`front`から`back`への通信は許可する。
- `public`の`front`から`private`の`front`への通信は許可する。
- `private`から`public`への通信を許可する。

2. 以下コマンドを実行し、通信結果が結果表の通りとなることを確認してください。（`142.250.191.142`はgoogle.comです。）

``` sh
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
```

**結果表**

|元|先|通信可否|
|-|-|-|
|front.public|back.public|○|
|front.public|front.private|○|
|front.public|back.private|✗|
|front.public|それ以外|✗|
|back.public|front.public|✗|
|back.public|front.private|✗|
|back.public|back.private|✗|
|back.public|それ以外|✗|
|front.private|front.public|○|
|front.private|back.public|○|
|front.private|back.private|○|
|front.private|それ以外|✗|
|back.private|front.public|○|
|back.private|back.public|○|
|back.private|front.private|✗|
|back.private|それ以外|✗|

3. Pod、NetworkPolicy、Namespaceを削除してください。

[*解答例*](../ans/networkpolicy.md)  

---

次： [metadataサーバへのアクセス制御](metadata-server.md)  
[トップ](../README.md)  
