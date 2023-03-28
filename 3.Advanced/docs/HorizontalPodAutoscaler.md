前： [MetricsServer](MetricsServer.md)  

---

# Horizontal Pod Autoscaler

Horizontal Pod Autoscaler（以下、HPA）はPodのHWリソース使用量を基にPodコントローラのreplica数を自動で増減させるリソースです。
HPAを動かすにはmetrics serverが必要です。（EKSの場合は利用者で手動デプロイ、AKS/GKEの場合はマネージドサービスでデプロイ済）
また、Pod（コンテナ）にresources.requestsを指定していることも条件になります。HPAは目標の負荷状態状態を宣言します。
この目標値に近づくようにPodの数量を変化させます。ターゲットとなるPod群の実際に消費しているHWリソース量の平均値をmetrics serverを使って計算します。
この計算した値を目標値で割り、どれくらいの数量のPodを追加/削除するか決定します。計算式で表すと以下の通りとなります。

```bash
desiredReplicas = ceil[currentReplicas * ( currentMetricValue / desiredMetricValue )]
```

# 演習

1. 以下を満たすマニフェストを作成しデプロイしてください。HPAについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - replica: 1
       - image:nginx: 1.12
       - resources.limits.cpu: 100m
     - Service
       - 上記Deploymentをtype:ClusterIPで公開
       - Portは80
     - HPA
       - 上記Deploymentを対象にする
       - Pod数を最小1 ~ 最大5の範囲とする
       - CPU使用率80%を目標値とする

1. 作成したオブジェクトを確認してください。Podが1つであることを確認してください。

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
     - Deployment
       - image: httpd
       - resources.limits.cpu: 100m
     - Service
       - 上記Deploymentをtype:ClusterIPで公開
       - Portは80
     - HPA
       - 上記Deploymentを対象にする
       - Pod数を最小1 ~ 最大5の範囲とする
       - CPU使用率80%を目標値とする

1. 上記作成したPodに以下の様な追加コマンドを発行してください。(宛先は作成したService名にする。最後の/を忘れずに。-nと-cの値はお好みで)

   ```bash
   ab -n 1000000 -c 100 http://httpd-svc/
   ```

1. 別ターミナルを開き、「kubectl top pod」と「kubectl get pod」をwatch等で監視し、以下を確認してください。
   - しばらくするとPodの数が増えること
   - Podが5つまで自動拡張されること
   - すべてのPodの負荷が80%(80m)を超えていてもPodが5つ以上増えないこと

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/HorizontalPodAutoscaler_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

---

次： [ClusterAutoscaler](ClusterAutoscaler.md)  
