前： [Pod-env](./Pod-env.md)

---

# 章末問題

最後に、初級で解説した内容をまとめた演習問題です。

## 問題1

次の要件を満たすmanifestを作成する。

- 要件
  - Deployment
    - Deploymentの名前は`practice`
    - replicas: `2`
    - Pod
      - labelはすべて`app: practice`
      - containerのイメージは`nginx:1.22`
      - manifestで指定する環境変数は`ENV1=Kono Container ha`、`ENV2=Nginx desu`
      - volumeMount
        - 下記で定義したhostPathのボリュームを指定する
        - コンテナのマウントパスは/usr/share/nginx/html/を指定する
    - volumes
      - `hostPath`のボリュームプラグインを使用
      - ホスト (worker node) の/mntなど任意のディレクトリを対象とする

  - Service
    - Serviceの名前は`practice-svc`
    - typeは`指定なし`（type: ClusterIPでも可）
    - Portは80
    - 上記Deployment:practiceで展開したPodを対象とする

## 問題2

1. 問題1で作成したmanifestをK8sクラスタ上にデプロイする。
1. デプロイしたPod内のコンテナに対して追加コマンドを発行し、/usr/share/nginx/html/index.htmlを以下内容に修正する。

   ```text
   zettai ni ushiatte ha ikenai data ga kochira dasu
   \$ENV1 \$ENV2
   ```

1. curlを実行できるPod:curlを作成する。
1. Service:practice-svcを経由してDeployment:practiceにアクセスする。
  （さきほど修正したindex.htmlが表示`される`ことを確認する）
1. Deployment:practiceのアクセスログを確認する。

## 問題3

1. Deployment:practiceのNginxのバージョンを`1.23`に`無停止で`アップデートする。
1. Pod:curlからService:practice-svcを経由してDeploymentにアクセスし続ける。
   - アップデート中にタイムアウトが発生しないことを確認する
   - 問題2で修正したindex.htmlが表示`される`ことを確認する。
1. Deployment:practiceとService:practice-svcを削除する。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Practice_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

---

次： [中級](../../2.Intermediate/README.md)
