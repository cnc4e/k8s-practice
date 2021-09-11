[トップ](../README.md)  
前： [アドミッションコントローラの追加](addmissioncontroller.md)  

---

# より安全なコンテナランタイムの使用

多くの場合、コンテナランタイムはDocker(runc)を使うことが多いと思います。しかし、Dockerよりも安全なコンテナランタイムもあります。たとえばgvisorなどがあります。K8sはワーカーノードごとにコンテナランタイムを自由に選ぶことができます。Podを動かしたいランタイムを指定できます。

1. ワーカーノードにgvisorをインストールしてください。([ヒント①](https://gvisor.dev/docs/user_guide/install/)、[ヒント②](https://sotoiwa.hatenablog.com/entry/2020/12/31/002527)）

2. ワーカーノードのcontainerdをgvisor(runsc)で使用するように構成してください。（[ヒント①](https://gvisor.dev/docs/user_guide/containerd/quick_start/)、[ヒント②](https://sotoiwa.hatenablog.com/entry/2020/12/31/002527)）

3. ワーカーノードのCRIをDockerからcontainerd(gvisor)に切り替えてください。（[ヒント](https://sotoiwa.hatenablog.com/entry/2020/12/31/002527)）

4. ノードの一覧を表示し、ランタイムがdockerからcotainerdに変わっていることを確認してください。

5. handlerとしてgvisor(runsc)を使用するRuntimeClassを作成してください。（([ヒント](https://kubernetes.io/ja/docs/concepts/containers/runtime-class/))）

6. 上記作成したRuntimeClassを使用してPodを起動してください。起動したPodに`dmseg`の追加コマンドを発行し、gvisorで動いていることを確認してください。（[ヒント](https://kubernetes.io/docs/concepts/containers/runtime-class/)）

7. PodとRuntimeClassを削除してください。

8. ワーカーノードのCRIをDcokerにもどしてください。

[*解答例*](../ans/runtimeclass.md)  

---

次： [イメージの脆弱性診断](image-scan.md)  
[トップ](../README.md)  
