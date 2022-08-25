前： [manifest](./manifest.md)

---

# APIリソースのカテゴリ

前章で

- APIリソースは、K8s上に作成される様々な機能/パーツのこと
- 複数のAPIリソースを組み合わせることで、K8s上にシステムを構築する

と解説しました。
本章以降は個々のAPIリソースの内容を解説していきます。

APIリソースは大きく次の5種類のカテゴリに分類されます。

|カテゴリ|概要|
|---|---|
|Workloads APIs|コンテナの管理と実行に関するリソース|
|Service APIs|コンテナへの接続・外部公開に関するリソース|
|Config & Storage APIs|設定や機密情報、外部ストレージに関するリソース|
|Metadata APIs|クラスタ内の他のリソースを操作するためのリソース|
|Cluster APIs|クラスタ本体の構成や定義|

各章で解説しているAPIリソースがどのカテゴリに所属しているのかを把握していると
情報の整理や公式マニュアルを参照する際などに役立ちますので、
今学んでいるAPIリソースがどのカテゴリなのか、常に意識していくと良いでしょう。

# Pod

PodはWorkloads APIsに分類されるAPIリソースの`最小単位`であり、`１つまたは複数のコンテナのグループ`です。  
Podは次の特徴を持ちます。

- 同じPodに含まれるコンテナは同一のIPアドレスを持つ。
- 同一のストレージを共有する。
- 必ず同一ノードで起動する。

> :warning:  
> 複数のコンテナを1つのPodにグループ化するのは、比較的高度なユースケースです。
> 基本的には`1Pod1コンテナ`と考えて問題ありません。
> 複数コンテナとする構成については、次のようなパターンがあります。
>
> - サイドカー
> - アンバサダー
> - アダプター
>
> 本章では細かく解説しませんが、こちらのキーワードは覚えておくとよいでしょう。

## チュートリアル1: Podを作成・操作する

では最初にkubectlの復習もかねて、Podに対する各種操作を一通り実施してみます。
次の操作を実施してください。
コマンドを確認したい場合は、[kubectl](./kubectl.md)を参照してください。

1. 次のmanifestを使用して、pod:testを作成する。

   ``` yml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test
     labels:
       app: test
   spec:
     containers:
       - image: nginx:1.12
         name: nginx
   ```

1. Podの一覧を参照し、pod:testを確認する。
1. pod:testの詳細情報を参照する。
1. pod:testの中に入って、`/usr/share/nginx/html/index.html`を参照する。
1. pod:testを削除する。

## チュートリアル2: 同じPod内で複数のコンテナを作成する

次に同じPod内に複数のコンテナを作成し、前述の特徴を確認します。
次の操作を実施してください。

1. 次のmanifestを使用して、pod:test2を作成する。

   ``` yml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test2
     labels:
       app: test2
   spec:
     containers:
       - image: nginx
         name: nginx
       - image: appropriate/curl
         name: curl
         command: ["sleep", "3600"]
   ```

1. Podの一覧を参照し、作成したPodを確認する。
1. 作成したPodの詳細情報を参照し、2つのコンテナが起動していることを確認する。
1. 各Podの中に入る。
   `-c <コンテナ名>`というオプションを追加することで、Pod内のどのコンテナに入るかを明示的にを指定することができる。

   ```bash
   kubectl exec -it test2 -c nginx -- /bin/sh
   kubectl exec -it test2 -c curl -- /bin/sh
   ```

1. 次のコマンドを打鍵して、各コンテナの`ホスト名`と`IPアドレス`が`一致していること`を確認する。

   ```bash
   # ホスト名の確認
   hostname
   # IPアドレスの確認
   hostname -i 
   ```

1. コンテナ`curl`で次のコマンドを打鍵する。

   ```bash
   curl localhost
   ```

1. pod:test2を削除する。

以上で本チュートリアルは終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Pod_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

---

次： [Deployment](Deployment.md)
