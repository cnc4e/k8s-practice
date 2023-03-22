前： [Pod-resources](Pod-resources.md)  

---

# Pod設定 livenessProbe/readinessProbe

K8sが通常チェックするPodの状態はコンテナのメインプロセスの有無のみです。（これはDocker Engine/Containerdの場合の話です。コンテナランタイムによっては異なる動作をするかもしれません。）
例えば、コンテナのメインプロセスがハングして正常に動作していない状態のコンテナを、K8sは異常と判定しませんし、自動復旧も行いません。
また、プロセスの起動後に初期化処理が必要なコンテナの場合、Pod起動（プロセス起動）と同時に通信を受けることになり、通信が失敗する可能性もあります。
これらを回避するため、コンテナのヘルスチェックを追加で設定することができます。ヘルスチェックはコンテナの`正常性確認（livenessProbe）`と`待受状態確認（readinessProbe）`の2種類があります。（K8s1.16以上ではさらに追加でstartupProbeも存在しますがこのpracticeでは触れません）
ヘルスチェックはPodに含まれるコンテナ単位で設定できます。

正常性確認（livenessProbe）はコンテナの稼働状態を確認するヘルスチェックです。このヘルスチェックに失敗した場合、コンテナの再作成が行われます。Podデプロイ後に威力を発揮するヘルスチェックです。  

待受状態確認（readinessProbe）もコンテナの稼働状態を確認するヘルスチェックです。ただし、こちらのヘルスチェックに失敗してもコンテナの再作成は行われません。代わりに、ヘルスチェックに失敗したコンテナを含むPodをServiceのバランシング対象Podから一時的に除外します。ヘルスチェックに成功するとServiceのバランシング対象Podにまた追加される。Podデプロイ時に威力を発揮するヘルスチェックです。

# 演習

1. 以下を満たすDeployment, Serviceをデプロイしてください。なお、probeについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - 名前は`nginx`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
         - readinessProbe
           - `httpGet`のヘルスチェックを行う
           - pathは`/`
           - portは`80`
           - ヘルスチェックの詳細設定はとくに行わない
     - Service
       - 名前は`nginx-svc`
       - 対象のlabelは`app: nginx`
       - プロトコルは`TCP`
       - Portは`80`
       - targetPortは`80`

1. Podリソースのオブジェクト一覧を表示し、上記マニフェストでデプロイしたPodの`READYが1/1`になっていることを確認してください。

1. Podの詳細を表示し、readinessProbeが設定されていることを確認してください。

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。（`応答があること`）

1. Service:nginx-svcの詳細を表示し、EndpointsにPodのIPアドレスが`設定されている`ことを確認してください。

1. Pod:nginxに以下の追加コマンドを発行してください。

   ```bash
   kill -SIGSTOP 7
   ```

1. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの`READYが0/1`になっていることを確認してください。

1. testpodから`Service:nginx-svc`に対して接続確認をしてください。（`応答がないこと`）

1. Service:nginx-svcの詳細を表示し、EndpointsにPodのIPアドレスが`設定されていない`ことを確認してください。

1. Pod:nginxに以下の追加コマンドを発行してください。

   ```bash
   kill -CONT 7
   ```

1. しばらく（10秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの`READYが1/1`になっていることを確認してください。。

1. testpodから`Service:nginx-svc`に対して接続確認してください。（`応答があること`）

1. Service:nginx-svcの詳細を表示し、EndpointsにPodのIPアドレスが`設定されている`ことを確認してください。

1. マニフェストを以下の内容に修正し、デプロイしてください。
    - readinessProbeのブロックを丸コピペし、readinessProbeを`livenessProbe`に書き換える
    - readinessProbeのブロックを`コメントアウト`する

1. Podリソースのオブジェクト一覧を確認し、Podの`RESTARTS`の値を確認してください。（たぶん0）

1. Podの詳細を表示し、livenessProbeが設定されていることを確認してください。

1. testpodから`Service:nginx-svc`に対して接続確認してください。（`応答があること`）

1. Pod:nginxに以下の追加コマンドを発行してください。

   ```bash
   kill -SIGSTOP 6
   ```

1. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの`RESTARTSが増えている`ことを確認してください。

1. testpodから`Service:nginx-svc`に対して接続確認してください。（`応答があること`）

1. マニフェストを以下の内容に修正し、デプロイしてください。
    - nginxのcommandを`['/bin/sh','-c','sleep 60;nginx -g "daemon off;"']`にする（nginxのプロセスがPod起動してから60秒後に起動する）
    - コメントアウトしていたreadinessProbeのブロックの`コメントアウトを解除`する
    - livenessProbeのヘルスチェックの詳細設定に`initialDelaySeconds: 120`を設定

1. （デプロイしてから60秒以内に実施）Podリソースのオブジェクト一覧を確認し、上記マニフェストでデプロイしたPodの`READYが0/1`になっていることを確認してください。

1. （デプロイしてから60秒以内に実施）testpodから`Service:nginx-svc`に対して接続確認をしてください。（`応答がないこと`）

1. （デプロイしてから60秒以降に実施）Podリソースのオブジェクト一覧を確認し、上記マニフェストでデプロイしたPodの`READYが1/1`になっていることを確認してください。

1. （デプロイしてから60秒以降に実施）testpodから`Service:nginx-svc`に対して接続確認をしてください。（`応答があること`）

1. Pod:nginxに以下の追加コマンドを発行してください。

   ```bash
   kill -SIGSTOP 9
   ```

1. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの`RESTARTSが増えている`ことを確認してください。

1. しばらく（60秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podのステータスが`READY 1/1`となっていることを確認してください。

1. testpodから`Service:nginx-svc`に対して接続確認をしてください。（`応答があること`）

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Pod-Probe_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
---
---

次： [Pod-initContainer](Pod-initContainer.md)  
