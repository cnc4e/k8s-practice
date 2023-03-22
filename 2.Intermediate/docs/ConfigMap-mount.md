前： [ConfigMap(env)](ConfigMap-env.md)  

---

# ConfigMap ファイルマウント

[前章](./ConfigMap-env.md)で触れましたが、ConfigMapをPodに紐づける方法はもう１つ`ConfigMapをVolumeにマウントする`方法があります。

例えばPod（コンテナ）内のとあるファイルの内容を頻繁に書き換えたい場合、コンテナイメージの中にそのファイルがあるとファイル内容を変えてからコンテナイメージをビルドしなおすことになります。
または、外部ストレージ(NFSサーバなど)にそのファイルを置いておき、initContainerなどでコンテナ起動前にコピーする方法もあります。
いずれの方法でも可能ですが、対象ファイルをConfigMapにすることでも可能です。
PodからConfigMapをマウントすることでPod外に設定ファイルを保存する事ができます。
ファイルの内容を変更したい場合はConfigMapを更新します。

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
     - ConfigMap
       - 以下の内容が記述されたindex.htmlをdataとして持つ
         - ConfigMap ni kaita naiyou dayo
     - Deployment
       - イメージはnginx:1.12
       - 上記ConfigMapをボリュームとしてマウント
       - マウント先は/usr/share/nginx/html
     - Service
       - 上記DeploymentをClusterIPのPort:80で公開

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。ConfigMapの内容が表示されることを確認してください。

1. ConfigMapのindex.htmlの内容を以下に修正し、ConfigMapを再デプロイしてください。
   - ConfigMap wo henkou suruto 60byou kuraide Pod nimo hanei sareruzo

1. （configmapをデプロイして60秒以内）curlが実行可能なPodからServiceを指定してcurlを実行してください。ConfigMap修正前の内容が表示されることを確認してください。

1. （configmapをデプロイして60秒以降）curlが実行可能なPodからServiceを指定してcurlを実行してください。ConfigMap修正後の内容が表示されることを確認してください。

1. 作成したリソースを削除してください。

なお、ボリュームマウントであるため、マウント先のディレクトリはConfigMap化したファイルのみになってしまいます。
もしも他のファイルがある場所にファイルを置きたい場合は、tempDirのボリュームを作り、initContainerで元のディレクトリ内容をtempDirにコピーする。
次にConfigMapもどこか別の場所にマウントしてからtempDirにコピー、メインコンテナでtempDirをしかるべき場所にマウントすれば実現できます。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/ConfigMap-mount_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

---

次： [Secret](Secret.md)  
