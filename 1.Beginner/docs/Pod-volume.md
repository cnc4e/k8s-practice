前： [Service-ClusterIP](Service-ClusterIP.md)

---

# volume

前章でも触れましたが、
K8s上に作成したPodは（永続的ではなく）比較的短命な存在であり、さまざまな理由で動的かつ頻繁に削除・再作成されます。
Podに割り当てられたディスク領域もまた一時的であり、Podが削除、再起動されるタイミングで、一緒に削除されます。

データベースやログ収集などを行うPod等の場合、ディスク領域が削除されるのは都合が悪いため、
Podが削除されてもデータを保持し続ける（データを永続化する）仕組みが提供されています。

この仕組みにはいくつか種類があります。
ここではファイルまたはディレクトリをワーカーノードのファイルシステムからマウントする方法について説明します。

## チュートリアル1: Pod再起動時にデータが失われることを確認する

まずは、Pod内に作成したデータがPodを削除・再起動することで失われることを確認してみましょう。

1. クラスタの参加ノードを確認しワーカーノードが`1台だけ`の状態にする。

1. 次のmanifestを使用して、DeploymentとServiceを作成する。

   ```yml
   kind: Service
   apiVersion: v1
   metadata:
     name: volume-svc
   spec:
     selector:
       app: volume
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```

   ```yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx
      labels:
        app: volume
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: volume
      template:
        metadata:
          labels:
            app: volume
        spec:
          containers:
          - name: nginx
            image: nginx:1.22
   ```

1. デプロイしたPod内のコンテナに対して追加コマンドを発行し、/usr/share/nginx/html/index.htmlを以下内容に修正する。

   ```text
   zettai ni nakunatte ha ikenai data ga kokoni aru
   ```

1. curlを実行できるPodを展開し、Service:volume-svcを経由してDeploymentにアクセスする。
  （さきほど修正したindex.htmlが表示`される`ことを確認する）

1. Podを削除（Deploymentは消さない！）し、Podが再作成されたことを確認する。

1. 再度Service:volume-svcを経由してDeploymentにアクセスする。
  （さきほど修正したindex.htmlが表示`されない`ことを確認する）

1. Deploymentを削除してください。（Serviceはそのままで良い）

## チュートリアル2: hostPathを使用することで、データが保持され続けることを確認する

次に、hostPathを使用して、Podが削除されてもデータが保持され続けることを確認してみましょう。

1. 次のmanifestを使用して、Deploymentを作成する。

   ```yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx
      labels:
        app: volume
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: volume
      template:
        metadata:
          labels:
            app: volume
        spec:
          containers:
          - name: nginx
            image: nginx:1.22
            volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: index.html
          volumes:
          - name: index.html
            hostPath:
              path: /mnt
              type: Directory
   ```

1. デプロイしたPod内のコンテナに対して追加コマンドを発行し、/usr/share/nginx/html/index.htmlを以下内容に修正する。

   ```text
   zettai ni nakunatte ha ikenai data ga kokoni aru
   ```

1. Service:volume-svcを経由してDeploymentにアクセスする。
  （さきほど修正したindex.htmlが表示`される`ことを確認する）

1. Podを削除（Deploymentは消さない！）し、Podが再作成されたことを確認する。

1. 再度1. Service:volume-svcを経由してDeploymentにアクセスする。
  （さきほど修正したindex.htmlが表示`される`ことを確認する）

1. Deployment:volume、Pod:curl、Service:volume-svcを削除してください。

以上で本チュートリアルは終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Pod-volume_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

---

次： [Pod-env](Pod-env.md)
