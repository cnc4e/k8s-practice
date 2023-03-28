前： [IngressController](IngressController.md)  

---

# Ingress

## http

Ingressを使い複数のサービスをhttpで公開します。

※ [IngressController](IngressController.md)実施直後の状態を想定しています。

1. 以下を満たすマニフェストを作成しデプロイしてください。Ingressリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - 1セット目
       - Deployment
         - コンテナイメージはnginx:1.12
         - /usr/share/nginx/html/index.htmlの内容を1つ目のアプリケーションであることがわかる内容に書き換える（内容自由）
       - Service
         - 上記DeploymentをClusterIPのPort:80で公開
       - Ingress
         - `test-1.k8s.practice`に対するアクセスのルール
         - バックエンドを上記ServiceのPort:80に指定
     - 2セット目
       - Deployment
         - コンテナイメージはnginx:1.12
         - /usr/share/nginx/html/index.htmlの内容を2つ目のアプリケーションであることがわかる内容に書き換える（内容自由）
       - Service
         - 上記DeploymentをClusterIPのPort:80で公開
       - Ingress
         - `test-2.k8s.practice`に対するアクセスのルール
         - バックエンドを上記ServiceのPort:80に指定

1. インターネットに接続可能でcurlが実行できる端末から以下コマンドを発行し、それぞれのServiceにIngressを経由してアクセスできていることを確認してください。  
  （プロキシ経由のアクセスだとリクエストがキャッシュされて表示が変わらないかもしれ。その場合はプロキシを通らない経路で試してみるとうまくいくかもしれない。）

   ```bash
   curl -H "Host:test-1.k8s.practice.local" http://<nginx ingress用LBのDNS名>
   curl -H "Host:test-2.k8s.practice.local" http://<nginx ingress用LBのDNS名>
   ```

   > :information_source:  
   > プロキシ経由のアクセスだとリクエストがキャッシュされて表示が変わらない可能性があります。
   > その場合はプロキシを経由しない通信経路で試してみると成功するかもしれません。

1. Nginx Ingress Controllerのログを確認し、上記2つのリクエストがIngress Controllerを経由していることを確認する。

1. 上記手順ではHTTPのリクエストヘッダにホスト名を入れることで接続しましたが。しかし本来であればホスト名である「test-1.k8s.practice」および「test-2.k8s.practice」をRoute53などのDNSにレコード追加して動作確認するべきです。以下を実行してください。
   1. VPC内部限定のプライベートホストゾーン「k8s.practice.local」を作成する
   1. 「*.k8s.practice.local」の宛先をNginx Ingress用LBとするCNAMEレコードを作成する。
   1. 以下のコマンドをVPC内のEC2インスタンス等から実行する。(Cloud9を使用するのが簡単でしょう。)

   ```bash
   curl http://test-1.k8s.practice.local
   curl http://test-2.k8s.practice.local
   ```

## https

続いてhttpsの場合も確認します。TLSの終端はIngress用のService Type:LBで行います。証明書は自己証明書を作成します。

1. 以下コマンドで自己証明書を作成してください。（CNが`*.k8s.practice.local`の自己証明書であれば、どのような方法で作成しても問題ありません。）

   ```bash
   openssl req -x509 -sha256 -nodes -newkey rsa:2048 -subj '/CN=*.k8s.practice.local' -keyout k8s.practice.local.key -out k8s.practice.local.crt
   ```

1. 作成した「k8s.practice.local.key」と「k8s.practice.local.crt」の内容を表示してください。（表示したテキストをACMに登録します。）

1. AWS ACMに自己証明書を登録してください。

1. （先の手順で使用した）VPC内のEC2に自己証明書のcrtファイルを`信頼されたルート証明書`として登録してください。

1. Nginx Ingress用のmanifestを修正し、TLSで使用する証明書を設定してください。（ヒント：証明書はACMのarnを指定します。）

1. 以下コマンドでhttpsで通信できることを確認してください。（`httpではない`）

   ```bash
   curl https://test-1.k8s.practice.local
   curl https://test-1.k8s.practice.local
   ```

1. 作成したリソースを削除してください。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Ingress_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/services-networking/ingress/

---

次： [NetworkPolicy](NetworkPolicy.md)  
