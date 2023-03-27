前： [descheduler](descheduler.md)  

---

# Ingress Controller

IngressはServiceの外部公開を制御するリソースおよびアドオン機能です。
L7ロードバランサの様なものです。Ingressはその動作をコントロールするIngress ControllerをPodとしてクラスタにアドオンします。
さらに、Ingress Controllerの動作設定を行うIngressリソース（こちらはK8sのリソース）を組み合わせて使います。

Kubernetes外からアクセスを受ける方法として、[Service Type:LB](../../2.Intermediate/docs/Service-LB.md)を紹介しました。
Service Type:LBはとても便利ですが問題もあります。それはServiceごとにクラウドのLBができてしまう点です。
たくさんのServiceを外部に公開したい場合、その数だけLBができてしまうのは管理、コストの面で問題となりえます。
また、他の外部公開の方法としてService Type:NodePortという手段もありますが、この公開方法だとServiceの公開ポートで同じ番号が使えず、よく使う80や443で公開できるServiceが限られてしまいます。

Ingressを使うとこれらの問題を解決する事ができます。
外部への公開はIngress用のService Type:LBが受け、Ingress Controllerに集約させます。
Ingress ControllerはリクエストのURLを見てしかるべきServiceにリクエストを転送します。
このURLと転送先Serviceの設定はIngressリソースで定義します。

Ingress Controllerにはいくつかの種類があります。その中でもNginx Ingress ControllerはK8sがサポートしており、どのクラウドでも利用可能です。

# 演習

1. Nginx Ingress Controllerをデプロイしてください。なお、[Nginx Ingress Controllerの公式][1]を参考にしてください。なお、Serviceは「service-l7.yaml」および「patch-configmap-l7.yaml」を使用してください。

1. AWSマネジメントコンソール（コマンドでも可）でNginx IngressのELBが作成されていることを確認してください。ELBのタグを見ればNginx Ingressのものか判断できる。

1. Nginx Ingress用LBにアタッチされているセキュリティグループを確認してください。

1. ワーカーノードのセキュリティグループを確認し、Nginx Ingress用LBのセキュリティグループからのインバウンドが許可されていることを確認してください。

次章[Ingress](Ingress.md)で実際の動作を確認しましょう。

具体的な操作およびその結果に関する回答例は[こちら](../ans/LimitRange_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.github.io/ingress-nginx/deploy/

---

次： [Ingress](Ingress.md)  
