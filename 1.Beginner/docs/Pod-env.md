前： [Pod-volume](Pod-volume.md)

---

# コンテナのメリット

コンテナ技術は可搬性の良さが特徴としてよく取り上げられます。

アプリケーション等の本番リリースは、一般的に次にような流れで行われます。

1. 開発環境を構築して開発を行う。
1. ステージング環境で動作検証・テストを実施する。
1. 本番環境へリリースする

コンテナの場合、アプリケーション本体に加えて、各種ライブラリなどのアプリケーション実行に必要な周辺環境を1つのコンテナイメージにまとめることで、
`アプリケーションの実行環境`をまるごと持ち運ぶことが可能になります。

これにより開発環境で構築したアプリケーション実行環境をそのままステージング環境や本番環境に移行することが簡単に行うことができ、
アプリケーション開発のリリース速度・ライフサイクルの迅速化ができる、というのがよく言われるコンテナのメリットです。

## 設定値の外だし

逆に言うと、コンテナ環境では同じコンテナイメージを各環境（開発/ステージング/本番等）で使いまわすことができるようにアプリケーションを実装するのが基本となります。
しかし、各環境ごとに設定値が異なるといったことは往々にしてあります。
例えば、本番と開発とではドメイン名が違う、外部DBのIPアドレスが違うなどです。

コンテナ環境においてこのような環境差異を表現するには環境ごとで変わる設定値を環境変数にし、コンテナ起動時に環境変数の値を設定するという手法で、
コンテナイメージの可搬性を維持します。

K8sでは、manifestの`spec`フィールドに環境変数を定義することが可能です。
この章ではこの機能について解説します。

## チュートリアル: 環境変数をmanifestに定義する

では実際にmanifestに環境変数を定義してみます。

1. 次のmanifestを使用して、Deployment:envを作成する。

    ``` yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: env
      labels:
        app: test
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: test
      template:
        metadata:
          labels:
            app: test
        spec:
          containers:
          - name: busybox
            image: busybox:1.35.0
            command: ['sh', '-c', 'echo $ENV1 $HOSTNAME $ENV2 && sleep 3600']
    ```

1. デプロイしたPod（コンテナ）のログを表示し、「<Pod名>」が表示されることを確認する。

1. デプロイしたPod（コンテナ）に追加コマンドを発行し設定されている環境変数を確認する。
   （環境変数の一覧は`printenv`というコマンドで確認できる。）

1. Deployment:envを削除する。

1. 次のmanifestを使用して、Deployment:envをデプロイする。

    ``` yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: env
      labels:
        app: test
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: test
      template:
        metadata:
          labels:
            app: test
        spec:
          containers:
          - name: busybox
            image: busybox:1.35.0
            command: ['sh', '-c', 'echo $ENV1 $HOSTNAME $ENV2 && sleep 3600']
            env:
            - name: ENV1
              value: "watashi no namae ha "
            - name: ENV2
              value: " desuYO"
    ```

1. デプロイしたPod（コンテナ）のログを表示し「<Pod名>」が表示されることを確認する。

1. デプロイしたPod（コンテナ）に追加コマンドを発行し設定されている環境変数を確認する。

1. Deployment:envを削除する。

以上で本チュートリアルは終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Pod-env_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

---

次： [演習問題](./Practice.md)
