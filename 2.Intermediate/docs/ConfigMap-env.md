前： [StatefulSet-volumeClaimTemplate](StatefulSet-volumeClaimTemplate.md)  

---

`ここまででだいぶK8sにも慣れてきたと思うのでここからは少し手順を簡略化して書いていきます。`

# ConfigMap

ConfigMapはPodに設定する環境変数やファイルをPodとは切り離して扱うようにするリソースです。
まずは環境変数としてのConfigMapの利用を確認します。

Podにパラメータを渡す方法として、[Podのenvに指定する方法](../../1.Beginner/docs/Pod-env.md)を紹介しましたが、1つ2つならまだしも、大量の環境変数が必要な場合に1つずつenvを定義するのは大変です。
このような場合、ConfigMapに設定情報を登録し、PodからConfigMapを読み込むことでパラメータを渡します。
また、ConfigMapは複数のPodに同じConfigMapを紐づけるができるため、複数Podで使用する環境変数の集中管理も可能です。

ConfigMapをPodに紐づける方法は次の2種類があります。

1. ConfigMapを環境変数として渡す
1. ConfigMapをVolumeにマウントする

まずは`ConfigMapを環境変数として渡す`方法を試してみます。

# 演習

1. 以下を満たすマニフェストを作成しデプロイしてください。ConfigMapについては[公式ドキュメント][1]を参考にしてください。PodでConfigMapを環境変数として読み込む方法は[公式ドキュメント][2]を参考にしてください。

   - 要件
     - ConfigMap
       - 以下のKey-Valueをdataとして持つ
         - EP1: The Python Menace
         - EP2: Attack of the Clones
         - EP3: Revenge of the Sith
         - EP4: A New Hoge
         - EF5: The Empire Strikes Back
         - EP6: Return of the Jedi
         - EP7: The Force Awakens
         - EP8: The Last Jedi
         - EP9: The Rise of Kubernetes
     - Deployment(1つ目)
       - メインコンテナはshが使用できれば何でも良い
       - 上記ConfigMapのKey-Valueをすべて環境変数として読み込む
       - メインコンテナのコマンドは以下を指定
         - command: `['/bin/sh','-c','echo "Star Wars no title ichiran \n $EP1 \n $EP2 \n $EP3 \n $EP4 \n $EP5 \n $EP6 \n $EP7 \n $EP8 \n $EP9"; sleep   3600']`
     - Deployment(2つ目)
       - メインコンテナはshが使用できれば何でも良い
       - 上記ConfigMapのKey-Valueをすべて環境変数として読み込む
       - メインコンテナのコマンドは以下を指定
         - command: `['/bin/sh','-c','echo "Star Wars no koukai jyun \n $EP4 \n $EP5 \n $EP6 \n $EP1 \n $EP2 \n $EP3 \n $EP7 \n $EP8 \n $EP9; sleep 3600']`

1. 各Deploymentで展開したPodのログを表示してください。

1. ConfigMapの値の誤りを修正し再デプロイしてください。なお、正しいvalueは[参考サイト][3]などを参照してください。  
  (誤りは2か所)

1. 各Deploymentで展開したPodのログを表示してください。表示が`変わらないこと`を確認してください。

1. 各Deploymentで展開したPodのログを削除し、セルフ・ヒーリングさせてください。

1. 各Deploymentで展開したPodのログを表示してください。表示が変わることを確認してください。

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/ConfigMap-env_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
[2]:https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables
[3]:https://dic.nicovideo.jp/a/%E3%82%B9%E3%82%BF%E3%83%BC%E3%82%A6%E3%82%A9%E3%83%BC%E3%82%BA

---

次： [ConfigMap(mount)](ConfigMap-mount.md)  
