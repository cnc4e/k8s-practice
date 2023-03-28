前： [HorizontalPodAutoscaler](HorizontalPodAutoscaler.md)  

---

# Cluster Autoscaler

Cluster Autoscaler（以下、CA）は自動でワーカーノード数を調整するアドオン機能です。
CAはK8s内にコントローラの役割を担うPodをデプロイして機能します。スケールアウトはHWリソースが足りずにどこにもスケジュールできないPod（STATUS:Pending）が発生した際に行われます。
スケールインはクラスタ全体のHWリソース割当量をみて行われます。なお、CAはEKS/AKS/GKEなど対応したクラウドプロバイダでのみ使用可能です。
また、ワーカーノードをAWS AutoScalingなどノード台数を柔軟に変更できる機能で展開していないと使用できません。

# 演習

1. ワーカーノードに付与されているIAMロールをマネジメントコンソール等で確認してください。以下のようにAutoScalingGroupの権限が設定されていることを確認してください。設定されていない場合はワーカーノードのIAMロールにポリシーを追加してください。([参考][1])

   ``` json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "autoscaling:DescribeAutoScalingGroups",
                   "autoscaling:DescribeAutoScalingInstances",
                   "autoscaling:DescribeLaunchConfigurations",
                   "autoscaling:SetDesiredCapacity",
                   "autoscaling:TerminateInstanceInAutoScalingGroup"
               ],
               "Resource": "*"
           }
       ]
   }
   ```

1. AutoScalingGroupのワーカーノードのタグをマネジメントコンソール等で確認してください。以下のタグが付与されていることを確認してください。付与されていない場合は付与してください。なお、タグはKeyの値が重要であり、valueは任意の値で問題ありません。

   - k8s.io/cluster-autoscaler/enabled
   - k8s.io/cluster-autoscaler/<クラスタ名>

1. [CAのマニフェスト][2]をコピーしてください。

1. 上記でダウンロードしたマニフェストの\<YOUR CLUSTER NAME\>の箇所を修正し、デプロイしてください。

1. ClusterAutoscalerのPodが起動していることを確認してください。

1. 以下を満たすマニフェストを作成しデプロイしてください。
   - Deployment
     - イメージは何でもよい
     - resource.limits.cpu: 1000m

1. 上記作成したDeploymentのreplica数を1つずつ増やし、STATUS:PendingのPodが出るまで続けてください。

1. 「kubectl get node」および「kubectl get pod」をwatch等で監視し、Nodeが増えてSTATUS:PendingのPodがrunningになることを確認してください。

   > :information_source:  
   >
   > - PendingのPodが作成されてからEC2インスタンスを立ち上げるため数分かかります。
   > - AutoScalingが正常に動作しない場合は、以下のサイトなどを参考にトラブルシューティングしてください。
   >   - [Amazon EKS クラスターで Cluster Autoscaler をセットアップするときの問題をトラブルシューティングする方法を教えてください。][3]
   > - その他、AutoScalingグループの`最大キャパシティ`などもチェックしてみると良いでしょう。
   >   - 最大キャパシティまでしかワーカーノードは作成されないため。

1. 上記作成したDeploymentのreplica数を1にしてください。

1. 「kubectl get node」および「kubectl get pod」をwatch等で監視し、Nodeが減ることを確認してください。

   > :information_source:  
   > デフォルトではスケールアウト後のスケールインは`10分以上`経過しないと行われません。
   > 経過時間が知りたい場合はpod/cluster-autoscalerのlogを確認してみると良いでしょう。`ノード名 was unneeded for 時間`で確認できます。
   > cf.) scale_down.go:829] ip-192-168-12-24.us-west-1.compute.internal was unneeded for 9m53.457855769s

1. 作成したリソースを削除してください。

以上で本演習は終了です。
具体的な操作およびその結果に関する回答例は[こちら](../ans/ClusterAutoscaler_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md
[2]:https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
[3]:https://repost.aws/ja/knowledge-center/amazon-eks-troubleshoot-autoscaler

---

次： [descheduler](descheduler.md)  
