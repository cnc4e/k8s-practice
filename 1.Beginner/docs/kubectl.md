前： -

---

# kubectl

Kubernetes(以降、K8s)には、`Control Plane`と呼ばれるK8sクラスタを操作・管理するコンポーネントが存在します。  
このControl Planeの１つに`kube-apiserver`があります。  
K8sの操作はこのkube-apiserverが外部に公開する`Kubernetes API`を利用し、`APIリソース`[^1]を登録することで行います。

> :information_source:  
> 本資料では、K8sの構造・構成に関する詳細は説明しません。  
> もし詳細を知りたい場合は、[公式ドキュメント: Kubernetesのコンポーネント][1]などを参照ください。

[1]:https://kubernetes.io/ja/docs/concepts/overview/components/

このKubernetes APIは`Restful API(http/https)`として実装されています。  
そのためcurlコマンドや各言語に存在するHTTPライブラリなどを使用して、K8sを操作することが可能です。

`kubectl`コマンドはmanifest[^2]、もしくはコマンドライン引数の情報を元に  
APIリクエストを自動生成し、K8sクラスタの操作を行うコマンドラインインターフェイスです。  
本章ではこのkubectlでよく使用する以下のサブコマンドの使い方について簡単に説明します。

|操作|コマンド|
| --- | --- |
|情報を取得・参照する     |kubectl get|
|APIリソースを作成する    |kubectl apply|
|詳細情報を取得・参照する  |kubectl describe|
|Pod内でコマンドを実行する |kubectl exec|
|Podのログを参照する      |kubectl logs|
|APIリソースを削除する    |kubectl delete|

> :information_source:  
> K8sを操作するためのコマンドラインインターフェイスは、kubectl以外にも、`kubeadm`が存在します。  
> kubeadmに関する詳細は[公式ドキュメント:Kubeadm][2]を参照ください。

> :information_source:  
> kubectl には上記以外にも様々なサブコマンドが存在します。  
> 詳細については[公式ドキュメント:kubectlの概要][3]を参照ください。

[2]:https://kubernetes.io/ja/docs/reference/setup-tools/kubeadm/
[3]:https://kubernetes.io/ja/docs/reference/kubectl/overview/

[^1]:K8s上に作成される様々な機能/パーツのこと
[^2]:K8sのAPIリソースに関する定義を記載した構造データ。多くはYAML形式で記述されます。

## チュートリアル1: kubectlを使ってみる

では実際にkubectlコマンドを使ってみましょう。  
まだkubectlをインストールしていない場合、[こちらのドキュメント][4]などを参照し、手元の環境にkubectlをインストールしてください。

[4]:https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/

次のコマンドを打鍵してください。

```bash
kubectl get node
```

次のような結果が表示されていれば成功です。

```bash
$ kubectl get node
NAME                                          STATUS   ROLES    AGE   VERSION
ip-192-168-2-239.us-west-1.compute.internal   Ready    <none>   8h    v1.22.6-eks-7d68063
ip-192-168-42-51.us-west-1.compute.internal   Ready    <none>   8h    v1.22.6-eks-7d68063
ip-192-168-42-65.us-west-1.compute.internal   Ready    <none>   8h    v1.22.6-eks-7d68063
```

> :warning:  
> エラーとなる場合は以下の観点で環境を確認してみてください。
>
> - NWの問題
>   - 先に説明した通り、kubectlはK8sクラスタに対してAPIリクエストを送信しています。
>   - timeoutなどのエラーとなる場合、NWに問題があり、K8sクラスタと正常に通信できていない可能性があります。
> - kubeconfigの問題
>   - kubectlでは、K8sクラスタへアクセスするための各種パラメータ（接続先URLや認証情報など）をkubeconfigファイルから取得しています。
>   - 認証などのエラーとなる場合、このkubeconfigの設定に問題がある可能性があります。

## チュートリアル2: kubectlとmanifestファイルからコンテナを起動する

次にkubectlとmanifestファイルからコンテナを起動してみましょう。

次の内容のファイルを作成してください。
ファイル名は任意の値でかまいません。ここではtutorial.yamlとします。
※manifestの内容については[次章](./manifest.md)で解説します。

```yml
apiVersion: v1
kind: Pod
metadata:
  name: test
  labels:
    app: test
spec:
  containers:
    - image: nginx:1.22
      name: nginx
```

manifestを使用してコンテナを作成・起動してみましょう。  
`kubectl apply -f <manifest>`で指定したmanifestからコンテナを作成・起動することができます。

次のコマンドを打鍵してください。

```bash
kubectl apply -f tutorial.yaml
```

以下の例のように`created`と表示されていれば成功です。

```bash
$ kubectl apply -f tutorial.yaml
pod/test created
```

## チュートリアル3: Pod一覧を取得する

起動したコンテナを確認してみましょう。  
`kubectl get <APIリソースの種別>`で指定したAPIリソースの一覧を取得できます。  
ここで指定している`Pod`はK8sにおいて最も基礎となるAPIリソースです。  
Podについては[別の章](./Pod.md)で解説しますので、ここでは`Pod = コンテナ`と考えてください。  
（以降はPodという名称で解説を進めます。）  

また`-o wide` を付け加えることにより、追加情報を取得できます。  
こちらはPodに割り当てられた`IPアドレスを確認するとき`、Podが稼働している`ノードを確認するとき`などに使用します。

次のコマンドを打鍵してください。

```bash
kubectl get pod
kubectl get pod -o wide
```

以下の例のようにPodの一覧が参照できます。

```bash
$ kubectl get pod
NAME   READY   STATUS    RESTARTS   AGE
test   1/1     Running   0          4m29s
$
$ kubectl get pod -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP              NODE                                          NOMINATED NODE   READINESS GATES
test   1/1     Running   0          11m   192.168.39.79   ip-192-168-42-51.us-west-1.compute.internal   <none>           <none>
```

## チュートリアル4: Podの詳細を確認する

作成したPodの詳細情報を確認しましょう。  
`kubectl describe <APIリソースの種別> <APIリソースの名前>`で詳細情報を確認できます。  
Podが正常起動しないなど、トラブルシュートの際などに使用します。

次のコマンドを打鍵してください。

```bash
kubectl describe pod test
```

次の例のようにpod/test の詳細情報が表示されます。

```bash
$ kubectl describe pod test
Name:         test
Namespace:    default
Priority:     0
Node:         ip-192-168-42-51.us-west-1.compute.internal/192.168.42.51
Start Time:   Mon, 16 May 2022 11:19:14 +0900
Labels:       app=test
Annotations:  kubernetes.io/psp: eks.privileged
Status:       Running
IP:           192.168.39.79
IPs:
  IP:  192.168.39.79
Containers:
  nginx:
    Container ID:   docker://7571ec8ace29fda9a55dce2e2709ae516be4fa3f168452b569ab10adbf624796
    Image:          nginx:1.22
    Image ID:       docker-pullable://nginx@sha256:72daaf46f11cc753c4eab981cbf869919bd1fee3d2170a2adeac12400f494728
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 16 May 2022 11:19:20 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-59685 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-59685:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  14m   default-scheduler  Successfully assigned default/test to ip-192-168-42-51.us-west-1.compute.internal
  Normal  Pulling    14m   kubelet            Pulling image "nginx:1.22"
  Normal  Pulled     14m   kubelet            Successfully pulled image "nginx:1.22" in 4.256753698s
  Normal  Created    14m   kubelet            Created container nginx
  Normal  Started    14m   kubelet            Started container nginx
```

## チュートリアル5: Podを更新する

Podの内容を更新してみましょう。  
先ほど使用したtutorial.yamlの`- image: nginx:1.22`を`- image: nginx:1.23`に修正し、
もう一度`kubectl apply`コマンドを実行します。

次のコマンドを打鍵してください。

```bash
kubectl apply -f tutorial.yaml
```

以下の例のように`configured`と表示されていれば成功です。

```bash
$ kubectl apply -f tutorial.yaml
pod/test configured
```

もう一度Podの詳細情報を確認しましょう。

次のコマンドを打鍵してください。

```bash
kubectl describe pod test
```

nginxのバージョンが1.23に変わっています。（★印）

このようにmanifestの内容を修正した場合は、初回起動時と同じく`kubectl apply`コマンドを実行することで修正を反映させることができます。

```bash
$ kubectl describe pod test
Name:         test
Namespace:    default
Priority:     0
Node:         ip-192-168-42-51.us-west-1.compute.internal/192.168.42.51
Start Time:   Mon, 16 May 2022 11:19:14 +0900
Labels:       app=test
Annotations:  kubernetes.io/psp: eks.privileged
Status:       Running
IP:           192.168.39.79
IPs:
  IP:  192.168.39.79
Containers:
  nginx:
    Container ID:   docker://7571ec8ace29fda9a55dce2e2709ae516be4fa3f168452b569ab10adbf624796
    Image:          nginx:1.23 ★
    Image ID:       docker-pullable://nginx@sha256:72daaf46f11cc753c4eab981cbf869919bd1fee3d2170a2adeac12400f494728
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 16 May 2022 11:19:20 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-59685 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-59685:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  14m   default-scheduler  Successfully assigned default/test to ip-192-168-42-51.us-west-1.compute.internal
  Normal  Pulling    14m   kubelet            Pulling image "nginx:1.22"
  Normal  Pulled     14m   kubelet            Successfully pulled image "nginx:1.22" in 4.256753698s
  Normal  Created    14m   kubelet            Created container nginx
  Normal  Started    14m   kubelet            Started container nginx
  Normal  Killing    9s    kubelet            Container nginx definition changed, will be restarted
  Normal  Pulled     9s    kubelet            Container image "nginx:1.23" already present on machine
```

## チュートリアル6: Pod内でコマンドを実行する

Pod内でコマンドを実行してみましょう。  
Podに対する操作は`kubectl exec`を使用します。  
構文は`kubectl exec <操作したいPod名> -- <実行するコマンド>`です。  
Podの状態確認や設定確認する場合などに使用します。  

次のコマンドを打鍵してください。

```bash
kubectl exec test -- cat /usr/share/nginx/html/index.html
```

この例では`Pod内`の/usr/share/nginx/html/index.htmlをcatコマンドで参照しています。

```bash
$ kubectl exec test -- cat /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

また`kubectl exec`を使用して、指定したPodの中に入るする（正確には実行中のコンテナへのシェルを取得する）ことができます。  
構文は`kubectl exec -it <操作したいpod名> -- /bin/sh`です。  
次の例では、pod/testの中に入り、curlコマンドを追加インストールしています。  

```bash
kubectl exec -it test -- /bin/bash
```

次の例ではcurlコマンドでnginxの動作確認を試みています。
その際にcurlコマンドがインストールされていなかったため、apt-getコマンドでcurlコマンドをインストールし、nginxの動作確認を実施しています。  
（次のチュートリアル7のために必要な操作となりますので、一緒にやってみてください。）
シェルから抜ける際は`exit`コマンドを使用します。

> :warning:  
> この操作は対象コンテナにシェル（/bin/shや/bin/bashなど）がインストールされている場合のみ可能です。
> シェルがインストールされていないコンテナイメージでは次のようなエラーで失敗します。
>
> ```bash
> OCI runtime exec failed: exec failed: container_linux.go:380: starting container > process caused: exec: "/bin/sh": stat /bin/sh: no such file or directory: unknown
> command terminated with exit code 126
> ```

> :warning:  
> インストールしたcurlコマンドは、一時的なものであり、永続的に反映されるものではありません。  
> このpod/testを再作成、削除すると消えてしまいますので、ご注意ください。

```bash
$ kubectl exec -it test -- /bin/bash
root@test:/#
root@test:/# curl localhost
bash: curl: command not found
root@test:/#
root@test:/# apt-get update
Ign:1 http://deb.debian.org/debian stretch InRelease
  (中略)
Get:9 http://nginx.org/packages/debian stretch/nginx amd64 Packages [25.8 kB]
Fetched 10.9 MB in 1s (8408 kB/s)
Reading package lists... Done
root@test:/#
root@test:/# apt-get install curl
Reading package lists... Done
Building dependency tree
  (中略)
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
root@test:/#
root@test:/# curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@test:/# exit
exit
```

## チュートリアル7: Podのログを参照する

Podのログを参照してみましょう。  
`kubectl logs`を使用します。  
構文は`kubectl logs <Pod名>`です。  

次のコマンドを打鍵してください。

```bash
kubectl logs test
```

次の例のように指定したPodから出力されるログが表示されます。  
この例ではチュートリアル3で実施したcurlコマンドによるhttpアクセスのログが表示されています。

```bash
$ kubectl logs test
127.0.0.1 - - [16/May/2022:01:35:31 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.52.1" "-"
```

## チュートリアル8: Podを削除する

最後に作成したPodを削除します。  
Podの削除は`kubectl delete`を使用します。

次のコマンドを打鍵してください。

```bash
kubectl delete -f tutorial.yaml
```

Podを削除した後、kubectl get で取得したPod一覧にpod/testが`存在しないこと`を確認してみましょう。

```bash
$ kubectl delete -f tutorial2.yaml
pod "test" deleted
$ 
$ kubectl get pod
No resources found in default namespace.
```

以上で本チュートリアルは終了です。

---

次： [manifest](manifest.md)
