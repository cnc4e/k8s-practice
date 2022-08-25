前： [Deployment](Deployment.md)

---

# Serviceとは？

K8s上に作成したPodは（永続的ではなく）比較的短命な存在であり、さまざまな理由で動的かつ頻繁に削除・再作成されます。
各Podは作成時にIPアドレスが動的に採番・割り当てされますが、このIPアドレスもPodの削除・再作成に合わせて変わります。

この仕組みだと、外部からPodにアクセスする際、Podに割り当てられたIPアドレスを毎回調査する必要があります。
この問題を解消するために用意されたAPIリソースが`Service`です。
こちらはService APIsのカテゴリに所属します。

> :warning:  
> Service APIsというカテゴリとAPIリソースのServiceは区別されますので注意してください。
> APIリソース：Serviceはカテゴリ：Service APIsに所属するAPIリソースの１つです。
>
> - Service APIs
>   - Service
>   - Ingress
>   - Endpoints
>   - ...

Serviceは単一のエンドポイントを提供し、受け取ったリクエストを指定したPodに転送してくれます。
Podへのアクセスを中継するL4ロードバランサのようなものです。

Serviceにはいくつかタイプがあり、代表的なものは次の通りです。
この章では基本となる`Cluster IP`について学びます。

- Cluster IP
- LoadBalancer
- NodePort
- ExternalName

## Cluster IP

Cluster IPはK8s`クラスタ内部でのみ通信できる`IPアドレスをエンドポイントとして設定するServiceです。
最も基本となるServiceとなります。

通信を転送するPodは、通常`ラベルセレクター`によって判別します。
以下はCluster IPのmanifestのサンプルです。
このサンプルでは、`app: test`というラベルを持ったPodやDeploymentに通信を転送します。(★印の部分)

```yml
kind: Service
apiVersion: v1
metadata:
  name: test-svc
spec:
  selector:
    app: test ★
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

## チュートリアル: Serviceを作成する

※前章`Deployment`実施直後の状態から開始してください。

では実際にCluster IPを作成してみましょう。
次の操作を実施してください。

1. 次のmanifestを使用して、Service:nginx-svcを作成する。

   ```yml
   kind: Service
   apiVersion: v1
   metadata:
     name: nginx-svc
   spec:
     selector:
       app: test
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```

1. 作成したService:nginx-svcに設定されたIPアドレスを確認する。

1. Serviceを経由してDeploymentにアクセスする。
   （宛先は先ほど確認した`Service:nginx-svcのIPアドレス`を指定する）

   > :information_source:  
   > 前述の通り、Cluster IPに外部から接続することはできません。
   > 今回は接続確認のためにcurlコマンドが入ったコンテナを含むPodを作成し、このPodでcurlコマンドを打鍵します。具体的な操作方法は以下を確認してください。
   >
   > ``` sh
   > # curl の起動
   > kubectl run curl --image=appropriate/curl -- /bin/sh -c "sleep 3600"
   > ```

1. Serviceを経由してDeploymentにアクセスする。
   （宛先は`nginx-svc`を指定する）

1. 2つあるPod:nginx-XXXXXそれぞれに含まれるtestコンテナに対して追加コマンドを発行し、コンテナ内の/usr/share/nginx/html/index.htmlを以下内容に修正する。

   > :information_source:  
   > nginxのコンテナイメージにはvi、nanoといったeditorが含まれていません。  
   > echo `文字列` > index.html などのコマンドで対象ファイルを修正しましょう。

   - どちらかのコンテナ

     ```text
     watashi no sentouryoku ha
     ```

   - もう一方のコンテナ

     ```text
     530000 desu
     ```

1. curlコマンドの宛先を`nginx-svc`とし、Serviceを経由してDeploymentに複数回アクセスする。
   (表示される結果が`ランダムで変わる`ことを確認する）

1. Service:nginx-svcのlabelSelectorを「app: test」から「`app: test2`」に修正し、修正を適用する。

1. curlコマンドの宛先を`nginx-svc`とし、Serviceを経由してDeploymentに複数回アクセスする。
   (`アクセスできない`ことを確認する）

1. Deployment:nginx、Deployment:curlおよびService:nginx-svcを削除する。

以上で本チュートリアルは終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Service-ClusterIP_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

---

次： [Pod-volume](Pod-volume.md)
