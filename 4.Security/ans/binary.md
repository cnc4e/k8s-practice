``` sh
wget https://dl.k8s.io/v1.22.0-rc.0/kubernetes-server-linux-amd64.tar.gz
tar -zxvf kubernetes-server-linux-amd64.tar.gz
cd ./kubernetes/server/bin

echo 3f547f39af1854edec1ef45229d1088eeb8a6d8e475d1533801f904d1119388b > kube-apiserver-sha256
echo 6c02eb05198b8ffa2feb5a1d6d52e3133d793c1b4d375ed498fd14aa70375847 > kube-controller-manager-sha256
echo dcebbc9275d30f71bfce4349352576c1b729c9d060b2dcd295b6dad1aa2e2c22 > kube-proxy-sha256
echo 883e9209612f9a1803a31fce10104288ac2ld78db40457e5deaed3912e6240d5 > kube-scheduler-sha256
echo 83f6534044f06b151f1857b507b1327331c3ba0eabf49e79bc04f272a721d5cf > kube-kubelet-sha256

sha256sum kube-apiserver >> kube-apiserver-sha256
sha256sum kube-controller-manager >> kube-controller-manager-sha256
sha256sum kubelet >> kube-kubelet-sha256
sha256sum kube-proxy >> kube-proxy-sha256
sha256sum kube-scheduler >> kube-scheduler-sha256

sed -i -e "s/ .*//g" kube-apiserver-sha256
sed -i -e "s/ .*//g" kube-controller-manager-sha256
sed -i -e "s/ .*//g" kube-kubelet-sha256
sed -i -e "s/ .*//g" kube-proxy-sha256
sed -i -e "s/ .*//g" kube-scheduler-sha256

uniq kube-apiserver-sha256
uniq kube-controller-manager-sha256
uniq kube-kubelet-sha256
uniq kube-proxy-sha256
uniq kube-scheduler-sha256

cd ../../../
rm -rf kubernetes kubernetes-server-linux-amd64.tar.gz
```

