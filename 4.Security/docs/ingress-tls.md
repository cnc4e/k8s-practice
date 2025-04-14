[トップ](../README.md)  
前： [kube-benchによる構成の検証](kube-bench.md)  

---

# IngressでTLSの終端

IngressはServiceを外部公開するリソースです。IngressでTLSの終端をすることもできます。

1. Ingress用の秘密鍵と証明書を作成。CNは`*`を指定すること。CAは何でも良い。（kube-apiserverで使用している認証局と同じものでよい。）

``` sh
openssl genrsa -out <秘密鍵> 4096
openssl req -new -key <秘密鍵> -out <証明書要求>
openssl x509 -req -days 9999 -in <証明書要求> -CA <CA証明書> -CAkey <CA秘密鍵> -CAcreateserial -out <証明書>
```

2. 作成した秘密鍵と証明書を使い、TLSタイプのSecretを作成してください。

3. nginx ingressをデプロイしてください。Ingress Controllerのserviceはtype:NodePortで公開されます。（[ヒント](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal)）

4. nginxのPodをデプロイしIngressで外部公開してください。パスは`/`でよいです。ingressClassNameはnginxを指定してください。なお、IngressでTLSの終端をしてください。（[ヒント](https://kubernetes.io/docs/concepts/services-networking/ingress/)）

5. 以下コマンドによりhttpsで接続できることを確認してください。

``` sh
curl -k https://<ノードIP or ドメイン名>:<Ingress NodePort>/
```

6. Pod,Service,Ingress,nginx-ingress,secretを削除してください。

[*解答例*](../ans/ingress-tls.md)  

---

次： [NetworkPolicyによるアクセス制御](networkpolicy.md)  
[トップ](../README.md)  
