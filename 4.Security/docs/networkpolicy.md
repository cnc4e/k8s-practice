[トップ](../README.md)  
前： [ingressでTLSの終端](ingress-tls.md)  

---

# NetworkPolicyによるアクセス制御

ネットワークポリシーはK8s内のSecurityGroupのようなものです。

## メタデータへのアクセス制限

メタデータはパブリッククラウドにおいてインスタンス等の情報です。このデータにアクセスすることでクラウド上のサーバ情報を知ることができます。Podが乗っ取られた場合を考えPodからメタデータへのアクセスを制限します。

1. nginxのPodを起動しそのPodから以下curlを実行しメタデータからホストの情報を取得できることを確認してください。（AWSの場合です。）

``` sh
curl -s http://169.254.169.254/latest/meta-data/
```

2. ネットワークポリシーを作成し、**メタデータへのアウトバウンドのみ**を制限してください。（ヒント：特定のIPのみ除外する場合は[except](https://kubernetes.io/docs/reference/kubernetes-api/policy-resources/network-policy-v1/)を使います。）

3. 再度nginxのPodからcurlを実行し、メタデータが取得できないことを確認してください。また、Podから他インターネット上のサービスにアクセスできることを確認してください。

4. PodおよびNetworkPolicyを削除してください。

## 複雑なアクセス制限

1. 以下を満たリソース群をデプロイしてください。

- Namespaceを2つ。名前は`public`と`private`
- 各Namespaceにはすべてのインバウンドとアウトバウンドを禁止するNetworkPolicyがついている
- 各Namespaceには2つのDeploymentがある。すべてreplica:1でnginxを動かす。一方のDeploymentには`app=front`、もう一方のDeploymentには`app=back`のラベルがついている。各Podの/usr/share/nginx/html/index.htmlはどのNamespaceのどのDeploymentか判別できる内容にする
- 各deploymentはService経由でport:80にアクセスできる
- 各Namespace内で`app=front`から`app=back`への通信は許可する。
- `public`の`app=front`から`private`の`app=front`への通信は許可する。
- `private`のすべてのDeploymentから`public`への通信を許可する。

|元|先|通信可否|
|-|-|-|
|public:app=front|public:app=back|○|
|public:app=front|private:app=front|○|
|public:app=front|private:app=back|✗|
|public:app=front|それ以外|✗|
|public:app=back|public:app=front|✗|
|public:app=back|private:app=front|✗|
|public:app=back|private:app=back|✗|
|public:app=back|それ以外|✗|
|private:app=front|private:app=back|○|
|private:app=front|public:app=front|○|
|private:app=front|public:app=back|○|
|private:app=front|それ以外|✗|
|private:app=back|private:app=front|✗|
|private:app=back|public:app=front|○|
|private:app=back|public:app=back|○|
|private:app=back|それ以外|✗|

[*解答例*](../ans/networkpolicy.md)  

---

次： [metadataサーバへのアクセス制御](metadata-server.md)  
[トップ](../README.md)  
