[トップ](../README.md)  
前： [ダッシュボードの強化](dashboard.md)  

---

# コンポーネントバイナリの検証

正しいバイナリを使用してコンポーネントを動かすことも大事です。巧妙に罠を仕掛けられた偽のバイナリで動かすと知らない内に危険にさらされているかもしれません。K8sのマスターコンポーネントにおいても同じです。信頼できる入手元のバイナリと一致しているか確かめる方法を実践します。

1. K8s v1.22のCHANGELOGにアクセスしてください。[こちら](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.22.md)です。

2. [Server Binaries](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.22.md#server-binaries)にある`kubernetes-server-linux-amd64.tar.gz`を端末にダウンロードしてください。

3. ダウンロードした`kubernetes-server-linux-amd64.tar.gz`を展開してください。

4. 展開したら`./kubernetes/server/bin`に移動してください。

5. 以下マスターコンポーネントのハッシュ値(SHA-256)を取得し、以下にかかれているハッシュ値と同じか確認してください。いくつかのコンポーネントはわざと違うハッシュ値を書いています。（ヒント：SHA-256のハッシュ値を得るにはsha256sumコマンドを使います。） 

|コンポーネント|比較用ハッシュ値(SHA-256)|
|-|-|
|kube-apiserver|3f547f39af1854edec1ef45229d1088eeb8a6d8e475d1533801f904d1119388b|
|kube-controller-manager|6c02eb05198b8ffa2feb5a1d6d52e3133d793c1b4d375ed498fd14aa70375847|
|kube-proxy|dcebbc9275d30f71bfce4349352576c1b729c9d060b2dcd295b6dad1aa2e2c22|
|kube-scheduler|883e9209612f9a1803a31fce10104288ac2ld78db40457e5deaed3912e6240d5|
|kubelet|83f6534044f06b151f1857b507b1327331c3ba0eabf49e79bc04f272a721d5cf|

6. `kubernetes-server-linux-amd64.tar.gz`および`kubernetes`ディレクトリを削除してください。

本来なら比較用のハッシュ値は信頼できる提供元にある値と比較します。今回はマスターコンポーネントのバイナリで実践しましたが、他のインターネットから入手したものは大体同じことが言えます。それらしいものもちゃんと提供元のハッシュ値と同じものか確認することでより安全と言えます。

[*解答例*](../ans/binary.md)  

---

次： [etcdの暗号化](etcd.md)  
[トップ](../README.md)  
