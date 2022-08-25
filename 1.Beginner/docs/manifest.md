前： [kubectl](./kubectl.md)

---

# manifestとは？

K8s上に作成される様々な機能/パーツのことを`APIリソース`と呼びます。
K8s上に構築されるシステムは複数のAPIリソースを組み合わせることで実現します。

`manifest/マニフェスト`はこのAPIリソースの管理に使用するファイルです。
具体的にはAPIリソースの構成をYAML形式で記述したテキストファイルです。

この章ではこのmanifestについて解説します。

## manifestの構造

manifestは次のフィールドで構成されます。

- apiVersion
- kind
- metadata
- spec

```yml
# manifestのサンプル
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
```

### apiVersion

このフィールドは`どのAPIグループ`の`どのバージョン`のAPIを使用するかを指定します。

フィールドの削除やリソース表現の再構成を簡単に行えるようにするため、Kubernetesは複数のAPIバージョンをサポートしており、
/api/v1や/apis/rbac.authorization.k8s.io/v1alpha1のように、それぞれ異なるAPIのパスが割り当てられています。
このフィールドで指定するAPIパスを切り替えることで簡単に複数のAPIバージョンの使い分けることができます。

例えば、`apps`という名前のAPIグループがあり、v1というバージョンが公開されています。
K8sの機能拡張などで、appsというAPIグループに所属するAPIに修正が行われ、v1beta1というバージョンで公開されたとします。
利用者はこのフィールドの指定をapps/v1からapps/v1beta1に変更するだけで、修正版のAPIバージョンに切り替えることができます。

実際に利用可能なAPIグループ/バージョンは[公式リファレンス][1]や`kubectl api-resources`というコマンドで確認することができます。

[1]:https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#-strong-api-groups-strong-

### kind

このフィールドは`どの種類のAPIリソース`を作成するかを指定します。

K8sにはpod、Deployment、Serviceなどの様々はAPIリソースが存在ます。
このフィールドでどのAPIリソースを作成するのかを指定します。

### metadata

このフィールドは`APIリソースを一意に特定するための情報`を指定します。

作成するAPIリソースにつける`name`や`label`といった情報を指定します。

### spec

このフィールドはkindで指定したAPIリソースのあるべき状態を指定します。
作成するAPIリソースによって、指定するフィールドは大きく変化します。

具体的なパラメータを知りたい場合は、以下のような方法で調べることができます。

1. 公式マニュアルやインターネット上で公開されているサンプルファイルなどを参照する
1. `kubectl explain`コマンドを使用する

例えばPodであれば、[PodSpec v1 core][1]に記載されています。

kubectl explainでは`kubectl explain pod.spec`と打鍵することで確認することができます。
複数のパラメータを設定する項目（例えばpod.spec.containersなど）の具体的なパラメータを知りたい場合、
`kubectl explain pod.spec.containers`のように打鍵することで確認することができます。

```bash
$ kubectl explain pod.spec
KIND:     Pod
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

     PodSpec is a description of a pod.

FIELDS:
   activeDeadlineSeconds        <integer>
     Optional duration in seconds the pod may be active on the node relative to
     StartTime before the system will actively try to mark it failed and kill
     associated containers. Value must be a positive integer.

   containers   <[]Object> -required-
     List of containers belonging to the pod. Containers cannot currently be
     added or removed. There must be at least one container in a Pod. Cannot be
     updated.

     (中略)

   topologySpreadConstraints    <[]Object>
     TopologySpreadConstraints describes how a group of pods ought to spread
     across topology domains. Scheduler will schedule pods in a way which abides
     by the constraints. All topologySpreadConstraints are ANDed.

   volumes      <[]Object>
     List of volumes that can be mounted by containers belonging to the pod.
     More info: https://kubernetes.io/docs/concepts/storage/volumes
```

```bash
$ kubectl explain pod.spec.containers
KIND:     Pod
VERSION:  v1

RESOURCE: containers <[]Object>

DESCRIPTION:
     List of containers belonging to the pod. Containers cannot currently be
     added or removed. There must be at least one container in a Pod. Cannot be
     updated.

     A single application container that you want to run within a pod.

FIELDS:
   args <[]string>
     Arguments to the entrypoint. The docker image's CMD is used if this is not
     provided. Variable references $(VAR_NAME) are expanded using the
     container's environment. If a variable cannot be resolved, the reference in
     the input string will be unchanged. Double $$ are reduced to a single $,
     which allows for escaping the $(VAR_NAME) syntax: i.e. "$$(VAR_NAME)" will
     produce the string literal "$(VAR_NAME)". Escaped references will never be
     expanded, regardless of whether the variable exists or not. Cannot be
     updated. More info:
     https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/#running-a-command-in-a-shell

   (中略)

   volumeDevices        <[]Object>
     volumeDevices is the list of block devices to be used by the container.

   volumeMounts <[]Object>
     Pod volumes to mount into the container's filesystem. Cannot be updated.

   workingDir   <string>
     Container's working directory. If not specified, the container runtime's
     default will be used, which might be configured in the container image.
     Cannot be updated.
```

[1]:https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#podspec-v1-core

以上で本チュートリアルは終了です。

---

次： [Pod](Pod.md)
