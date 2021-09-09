[トップ](../README.md)  
前： [PodSecurityPolicyによるポリシー定義](podsecuritypolicy.md)  

---

# アドミッションコントローラの追加

アドミッションコントローラはkube-apiserverへのリクエストが認証・認可された後、リクエストの内容を変更または検証を行うプラグインです。アドミッションコントローラは`validating`と`mutating`の2種類あり、それぞれ`検証`と`変更`ができます。これらのコントローラはkube-apiserverの設定を変更せず動的に追加できます。それをするにはアドミッションwebhookサーバを構成します。アドミッションwebhookサーバはK8s内でも外部でも建てられます。

validatingのアドミッションコントローラとしてk8sリソースのポリシーを定義、チェックすることができるGatekeeperをデプロイし動作を確認します。

1. OPAをアドミッションコントローラとして追加できるGatekeeperをデプロイしてください。（[ヒント](https://open-policy-agent.github.io/gatekeeper/website/docs/install)）

2. 以下を満たすConstraintTemplateおよびConstraintsをそれぞれ定義してください。（[ヒント①](https://open-policy-agent.github.io/gatekeeper/website/docs/howto)、[ヒント②](https://github.com/open-policy-agent/gatekeeper-library)）

- deploymentのreplicasは3以下でないとダメ
- podのimageはbitnamiのものじゃないとダメ

3. 以下のDeploymentをデプロイしてください。ポリシーに違反しデプロイできないことを確認してください。

- image: nginx
- replicas: 4

4. 以下のDeploymentをデプロイしてください。Deploymentはcreateできますが、Replicasetを確認しPodがデプロイできないことを確認してください。

- image: nginx
- replicas: 3

5. 以下のDeploymentをデプロイしてください。Podがデプロイできていることを確認してください。

- image: bitnami/nginx
- replicas: 3

6. Deployment、ConstraintTemplate、Constraintsを削除してください。

7. Gatekeeperを削除してください。

[*解答例*](../ans/addmissioncontroller.md)  

---

次： [より安全なコンテナランタイムの使用](runtimeclass.md)  
[トップ](../README.md)  
