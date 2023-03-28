# 回答例

## http

1. 以下を満たすマニフェストを作成しデプロイしてください。Ingressリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - 1セット目
       - Deployment
         - コンテナイメージはnginx:1.12
         - /usr/share/nginx/html/index.htmlの内容を1つ目のアプリケーションであることがわかる内容に書き換える（内容自由）
       - Service
         - 上記DeploymentをClusterIPのPort:80で公開
       - Ingress
         - `test-1.k8s.practice`に対するアクセスのルール
         - バックエンドを上記ServiceのPort:80に指定
     - 2セット目
       - Deployment
         - コンテナイメージはnginx:1.12
         - /usr/share/nginx/html/index.htmlの内容を2つ目のアプリケーションであることがわかる内容に書き換える（内容自由）
       - Service
         - 上記DeploymentをClusterIPのPort:80で公開
       - Ingress
         - `test-2.k8s.practice`に対するアクセスのルール
         - バックエンドを上記ServiceのPort:80に指定

   【回答例】

   ```yml
   # manifest
   ## 1セット目
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-1
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: nginx-1
     template:
       metadata:
         labels:
           app: nginx-1
       spec:
         containers:
           - name: nginx-1
             image: nginx:1.12
             lifecycle:
               postStart:
                 exec:
                   command:
                     ["/bin/sh", "-c", "echo $HOSTNAME >> /usr/share/nginx/html/index.html"]
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-1-svc
   spec:
     ports:
       - name: "http-port"
         protocol: "TCP"
         port: 80
         targetPort: 80
     selector:
       app: nginx-1
   ---
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: nginx-1-ingress
   spec:
     ingressClassName: nginx
     rules:
       - host: test-1.k8s.practice
         http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: nginx-1-svc
                 port:
                   number: 80
   ```

   ```yml
   # manifest
   ## 2セット目
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-2
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: nginx-2
     template:
       metadata:
         labels:
           app: nginx-2
       spec:
         containers:
           - name: nginx-2
             image: nginx:1.12
             lifecycle:
               postStart:
                 exec:
                   command:
                     ["/bin/sh", "-c", "echo $HOSTNAME >> /usr/share/nginx/html/index.html"]
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-2-svc
   spec:
     ports:
       - name: "http-port"
         protocol: "TCP"
         port: 80
         targetPort: 80
     selector:
       app: nginx-1
   ---
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: nginx-2-ingress
   spec:
     ingressClassName: nginx
     rules:
       - host: test-2.k8s.practice.local
         http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: nginx-2-svc
                 port:
                   number: 80
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f Ingress-nginx-1.yaml
   deployment.apps/nginx-1 created
   service/nginx-1-svc created
   ingress.networking.k8s.io/nginx-1-ingress created

   $ kubectl apply -f Ingress-nginx-2.yaml
   deployment.apps/nginx-2 created
   service/nginx-2-svc created
   ingress.networking.k8s.io/nginx-2-ingress created
   ```

1. インターネットに接続可能でcurlが実行できる端末から以下コマンドを発行し、それぞれのServiceにIngressを経由してアクセスできていることを確認してください。  
  （プロキシ経由のアクセスだとリクエストがキャッシュされて表示が変わらないかもしれ。その場合はプロキシを通らない経路で試してみるとうまくいくかもしれない。）

   ```bash
   curl -H "Host:test-1.k8s.practice.local" http://<nginx ingress用LBのDNS名>
   curl -H "Host:test-2.k8s.practice.local" http://<nginx ingress用LBのDNS名>
   ```

   > :information_source:  
   > プロキシ経由のアクセスだとリクエストがキャッシュされて表示が変わらない可能性があります。
   > その場合はプロキシを経由しない通信経路で試してみると成功するかもしれません。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get ingress
   NAME              CLASS   HOSTS                       ADDRESS                                                                         PORTS   AGE
   nginx-1-ingress   nginx   test-1.k8s.practice.local   a3709aeb2b22b45f182e39b79f8072bd-7d460318b6b03082.elb.us-west-1.amazonaws.com   80      2d21h
   nginx-2-ingress   nginx   test-2.k8s.practice.local   a3709aeb2b22b45f182e39b79f8072bd-7d460318b6b03082.elb.us-west-1.amazonaws.com   80      2m34s

   $ kubectl get pod
   NAME                      READY   STATUS      RESTARTS   AGE
   nginx-1-f57b57756-7zlfd   1/1     Running     0          58s
   nginx-2-68d8c9ff8-8b96q   1/1     Running     0          41s

   $ curl -H "Host:test-1.k8s.practice.local" http://ae845e18a3c78409d8fc4889be8cbfed-63b65b447a673f75.elb.us-west-1.amazonaws.com
   nginx-1-f57b57756-7zlfd

   $ curl -H "Host:test-2.k8s.practice.local" http://ae845e18a3c78409d8fc4889be8cbfed-63b65b447a673f75.elb.us-west-1.amazonaws.com
   nginx-2-68d8c9ff8-8b96q
   ```

1. Nginx Ingress Controllerのログを確認し、上記2つのリクエストがIngress Controllerを経由していることを確認する。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod -n ingress-nginx
   NAME                                        READY   STATUS      RESTARTS   AGE
   ingress-nginx-admission-create-549pl        0/1     Completed   0          7m37s
   ingress-nginx-admission-patch-c9m6r         0/1     Completed   0          7m37s
   ingress-nginx-controller-868bd5467d-2sshw   1/1     Running     0          7m38

   $ kubectl logs ingress-nginx-controller-868bd5467d-2sshw -n ingress-nginx
   -------------------------------------------------------------------------------
   NGINX Ingress controller
     Release:       v1.6.4
     Build:         69e8833858fb6bda12a44990f1d5eaa7b13f4b75
     Repository:    https://github.com/kubernetes/ingress-nginx
     nginx version: nginx/1.21.6

   -------------------------------------------------------------------------------

     (略)

   54.176.119.132 - - [27/Mar/2023:08:21:32 +0000] "GET / HTTP/1.1" 200 24 "-" "curl/7.81.0" 88 0.003 [default-nginx-1-svc-80] [] 192.168.6.215:80 24 0.003 200 0e51b8d3d144072f
   c351bf966e9b5cdf
   54.176.119.132 - - [27/Mar/2023:08:22:18 +0000] "GET / HTTP/1.1" 200 24 "-" "curl/7.81.0" 88 0.001 [default-nginx-2-svc-80] [] 192.168.61.126:80 24 0.000 200 767b673d37ded27
   23f1fb22fe719c3fd
   ```

1. 上記手順ではHTTPのリクエストヘッダにホスト名を入れることで接続しましたが。しかし本来であればホスト名である「test-1.k8s.practice」および「test-2.k8s.practice」をRoute53などのDNSにレコード追加して動作確認するべきです。以下を実行してください。
   1. VPC内部限定のプライベートホストゾーン「k8s.practice.local」を作成する
   1. 「*.k8s.practice.local」の宛先をNginx Ingress用LBとするCNAMEレコードを作成する。
   1. 以下のコマンドをVPC内のEC2インスタンス等から実行する。(Cloud9を使用するのが簡単でしょう。)

   ```bash
   curl http://test-1.k8s.practice.local
   curl http://test-2.k8s.practice.local
   ```

   【回答例】

   ![create hostzone](../images/create_hostzone.png)
   ![create cname](../images/create_cname.png)

   ```bash
   # 実行結果
   $ curl http://test-1.k8s.practice.local
   nginx-1-f57b57756-7zlfd

   $ curl http://test-2.k8s.practice.local
   nginx-2-68d8c9ff8-8b96q
   ```

## https

1. 以下コマンドで自己証明書を作成してください。（CNが`*.k8s.practice.local`の自己証明書であれば、どのような方法で作成しても問題ありません。）

   ```bash
   openssl req -x509 -sha256 -nodes -newkey rsa:2048 -subj '/CN=*.k8s.practice.local' -keyout k8s.practice.local.key -out k8s.practice.local.crt
   ```

   【回答例】

   ```bash
   $ openssl req -x509 -sha256 -nodes -newkey rsa:2048 -subj '/CN=k8s.practice.local' -keyout k8s.practice.local.key -out k8s.practice.local.crt
   Generating a RSA private key
   .........................................................+++++
   ..............................+++++
   writing new private key to 'k8s.practice.local.key'
   -----
   ```

1. 作成した「k8s.practice.local.key」と「k8s.practice.local.crt」の内容を表示してください。（表示したテキストをACMに登録します。）

   【回答例】

   ```bash
   # 実行結果
   $ cat k8s.practice.key
   -----BEGIN PRIVATE KEY-----
   MIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQDkx2/qC5NF85SB
   Ykd4HpGHocwI50XBTdYDf+e3CiYxah+eoG4mq0+SdY3ZEzGx4X+aFhT/NqY1t14H
   1oOdrgXzfLcP+KWO5Os9E+6kRN+WJnhsXG4EHHD01Ng1sgLvqVLeEkDdQW8nSqhe
   PnUElpmwVYjBS5PIDybh2hqXZMk3KMZ1nJQy+GAMmyms2KsnyIZFJb137lt/6p+L
   IR0bInU24sK+23hxGjqUDywc3hGbAIaUZZLbYMSkC+EU07mcbEu/53h+9ZBdSqUO
   nLrp1fk53syEchXAvMKibYy4v3OYEz6G7rHbgsWyjMGflK3WOwqEYdDTzr0Wocld
   maZ0nUYlAgMBAAECggEACNXW8Z/+0Q3GJ3UHM5Fip21NNu7Pl9aMtFflOAFxVik2
   DWc+ucmV6h8WGlMOcUM3LVk6+FxA8QQi4oSbUqSc4DlpzHka07jtiM9TQQ7T37We
   oviy2v/KMPpjWzeDu4O7io4D6UeUplzQwcsBs+qpBtFW1bg4fdpv8vr8Wepvhuy+
   gla2ji6VXXFEd6AOgmD7gcuC7HN9UBdMJbXW17Ujc4ouWTKZYaIU6RZMnLXmlXg3
   LFnZsTZ8gnRXZcL6RBiFZx5SCZK33LAw7Zv6TO0log09mz/HCjY1qvpfpjCrEll0
   qOxBXNspYIC8VYcRyxFXHTEBocxRcMndn/yVpYQ94QKBgQD4k/im+PWTuZ4+rtVx
   dt4AiV4Q/pMmwZg2fqEg3GXDSxn61VFHfVDB1aJDTp3GRKQqc8KGlltlpt+MDctL
   rS/LPcBv1Nrj3lUE3b8ScDtbFt2YLv56kUm8pzHfXDn+J05/KMtqVSng3cUORuNC
   HOlWXvsTu9Y6eXd2HAT8kF1aOQKBgQDrnCF1lO1MxtZKFIPeg8FeG1xi8eFZZpEY
   vQAuqVi/NCQGlZgXPN5aldhBaIw5mXDT06meN3MKicfoyoKCda49lhsvHFKQPDKn
   zAhzr8XZ/rTF1ULTHrwp+GWW2ZtHnqKpE83iKlpJTYlpR/aScuh+YkdmvCKp8ufu
   h6dPwq47TQKBgDK5FwsTUCE1ximqSVme1MwtO6miGOrWpDWyfzSSZJ2GLKX3c19w
   HdN/wV2UpDM5/HnBJ1eJRNsoQwN0KSCnFVKWpjbrq0LGMnkMupmLl0EOGGeeZ0Uh
   cndxcobObfMDmEvwShnHiQUem8A/DM6H0cFTHk63jI5gDNVoAEh9ON9ZAoGAXXbs
   VwajNEfUjkKhJU5DER/tgFrPuHVVreib/+ayC/eVKv+A/AbrdifnR9s/dUIHXiCj
   Mt5wRoftw81CBJrGaiv30LJ9ljYrKiFoYOwjopzF/YziG3ofzs55DmR4kuq9QDFZ
   BRUkB9w4XIltqR7oV+qIxJpxbdWsaBVsK9ng4CkCgYAa2L6348F6KpAtZsay9Z5S
   DbGmXQiidF8xO0FVbqEIXFnaHo0L9RPqEUw8e+gjhtbk3gTu7E8U3u7bxpg9NkAe
   LuVX0RrmxEcw4dYs6CWdwu9GDixWU1fAnGsS75ppQD88Wc0Ostqoex6y+S7F+SfO
   EvY0jdF2dZ8K1cpT6tYHaw==
   -----END PRIVATE KEY-----

   $ cat k8s.practice.local.crt
   -----BEGIN CERTIFICATE-----
   MIIDHzCCAgegAwIBAgIUZwVmhZvUSRhJcXPZ+crguYltCgQwDQYJKoZIhvcNAQEL
   BQAwHzEdMBsGA1UEAwwUKi5rOHMucHJhY3RpY2UubG9jYWwwHhcNMjMwMzI3MDYw
   OTE5WhcNMjMwNDI2MDYwOTE5WjAfMR0wGwYDVQQDDBQqLms4cy5wcmFjdGljZS5s
   b2NhbDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAOTHb+oLk0XzlIFi
   R3gekYehzAjnRcFN1gN/57cKJjFqH56gbiarT5J1jdkTMbHhf5oWFP82pjW3XgfW
   g52uBfN8tw/4pY7k6z0T7qRE35YmeGxcbgQccPTU2DWyAu+pUt4SQN1BbydKqF4+
   dQSWmbBViMFLk8gPJuHaGpdkyTcoxnWclDL4YAybKazYqyfIhkUlvXfuW3/qn4sh
   HRsidTbiwr7beHEaOpQPLBzeEZsAhpRlkttgxKQL4RTTuZxsS7/neH71kF1KpQ6c
   uunV+TnezIRyFcC8wqJtjLi/c5gTPobusduCxbKMwZ+UrdY7CoRh0NPOvRahyV2Z
   pnSdRiUCAwEAAaNTMFEwHQYDVR0OBBYEFPOmd0XJtUMqXfin/LhMZl1b4VjIMB8G
   A1UdIwQYMBaAFPOmd0XJtUMqXfin/LhMZl1b4VjIMA8GA1UdEwEB/wQFMAMBAf8w
   DQYJKoZIhvcNAQELBQADggEBANPLjqvL+raPHbVDtlp0IWdgojAjTBLuDbWSkg5G
   X3Aid1seOmp1U/lsEk9sR3db0+h84uGNmbfDW3HLmTX+K+DQUrCbIWkgDvQNBm8R
   I8k/iS0wDkNsJAVxPVEtCCFOgDTxIcemTZ8KaYrUk4Fh3nBpckxoJOP2NrebvrOY
   roHGDQDrhYFo/vp/dcaYlB5wvBh4AbADe+D3qW+Js3FlcYEJha9kgcitsKlmMRol
   NuD5ix3eaitcwDN9atDP0Ym0Gb+mETUIIfEH1BSQm3Miuxzgxxm/VVFsd+AlZ7av
   3muPoBb1sr6bY8RhPO93woKHLli8DnX0yk+NXS69S4GGnaQ=
   -----END CERTIFICATE-----
   ```

1. AWS ACMに自己証明書を登録してください。

   【回答例】

   ![import certificate 1](../images/import_certificate_1.png)
   ![import certificate 2](../images/import_certificate_2.png)

1. （先の手順で使用した）VPC内のEC2に自己証明書のcrtファイルを`信頼されたルート証明書`として登録してください。

   【回答例】

   ```bash
   # 実行結果
   $ sudo cp -p k8s.practice.local.crt  /etc/pki/ca-trust/source/anchors/
   $ sudo update-ca-trust extract
   ```

1. Nginx Ingress用のmanifestを修正し、TLSで使用する証明書を設定してください。（ヒント：証明書はACMのarnを指定する。）

   【回答例】

   ```yml
   # manifest
   ## 変更があるmanifestのみ抜粋。
    (略)
   ---
   apiVersion: v1
   data:
     allow-snippet-annotations: "true"
     http-snippet: |
       server {
         listen 2443;
         return 308 https://$host$request_uri;
       }
     proxy-real-ip-cidr: 192.168.0.0/16
     use-forwarded-headers: "true"
   kind: ConfigMap
   metadata:
     labels:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-controller
     namespace: ingress-nginx
   ---
   apiVersion: v1
   kind: Service
   metadata:
     annotations:
       service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"
       service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
       service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:us-west-1:456247443832:certificate/4fcb3c48-8d0d-4e0c-b2f6-49bfdc8753bd
       service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
       service.beta.kubernetes.io/aws-load-balancer-type: nlb
     labels:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-controller
     namespace: ingress-nginx
   spec:
     externalTrafficPolicy: Local
     ipFamilies:
       - IPv4
     ipFamilyPolicy: SingleStack
     ports:
       - appProtocol: http
         name: http
         port: 80
         protocol: TCP
         targetPort: tohttps
       - appProtocol: https
         name: https
         port: 443
         protocol: TCP
         targetPort: http
     selector:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
     type: LoadBalancer
   ---
    (略)
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-controller
     namespace: ingress-nginx
   spec:
     minReadySeconds: 0
     revisionHistoryLimit: 10
     selector:
       matchLabels:
         app.kubernetes.io/component: controller
         app.kubernetes.io/instance: ingress-nginx
         app.kubernetes.io/name: ingress-nginx
     template:
       metadata:
         labels:
           app.kubernetes.io/component: controller
           app.kubernetes.io/instance: ingress-nginx
           app.kubernetes.io/name: ingress-nginx
       spec:
         containers:
           - args:
               - /nginx-ingress-controller
               - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
               - --election-id=ingress-nginx-leader
               - --controller-class=k8s.io/ingress-nginx
               - --ingress-class=nginx
               - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
               - --validating-webhook=:8443
               - --validating-webhook-certificate=/usr/local/certificates/cert
               - --validating-webhook-key=/usr/local/certificates/key
             env:
               - name: POD_NAME
                 valueFrom:
                   fieldRef:
                     fieldPath: metadata.name
               - name: POD_NAMESPACE
                 valueFrom:
                   fieldRef:
                     fieldPath: metadata.namespace
               - name: LD_PRELOAD
                 value: /usr/local/lib/libmimalloc.so
             image: registry.k8s.io/ingress-nginx/controller:v1.6.4@sha256:15be4666c53052484dd2992efacf2f50ea77a78ae8aa21ccd91af6baaa7ea22f
             imagePullPolicy: IfNotPresent
             lifecycle:
               preStop:
                 exec:
                   command:
                     - /wait-shutdown
             livenessProbe:
               failureThreshold: 5
               httpGet:
                 path: /healthz
                 port: 10254
                 scheme: HTTP
               initialDelaySeconds: 10
               periodSeconds: 10
               successThreshold: 1
               timeoutSeconds: 1
             name: controller
             ports:
               - containerPort: 80
                 name: http
                 protocol: TCP
               - containerPort: 80
                 name: https
                 protocol: TCP
               - containerPort: 2443
                 name: tohttps
                 protocol: TCP
               - containerPort: 8443
                 name: webhook
                 protocol: TCP
             readinessProbe:
               failureThreshold: 3
               httpGet:
                 path: /healthz
                 port: 10254
                 scheme: HTTP
               initialDelaySeconds: 10
               periodSeconds: 10
               successThreshold: 1
               timeoutSeconds: 1
             resources:
               requests:
                 cpu: 100m
                 memory: 90Mi
             securityContext:
               allowPrivilegeEscalation: true
               capabilities:
                 add:
                   - NET_BIND_SERVICE
                 drop:
                   - ALL
               runAsUser: 101
             volumeMounts:
               - mountPath: /usr/local/certificates/
                 name: webhook-cert
                 readOnly: true
         dnsPolicy: ClusterFirst
         nodeSelector:
           kubernetes.io/os: linux
         serviceAccountName: ingress-nginx
         terminationGracePeriodSeconds: 300
         volumes:
           - name: webhook-cert
             secret:
               secretName: ingress-nginx-admission
   ---
    (略)
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f Ingress-deploy.yaml
   namespace/ingress-nginx unchanged
   serviceaccount/ingress-nginx unchanged
   serviceaccount/ingress-nginx-admission unchanged
   role.rbac.authorization.k8s.io/ingress-nginx unchanged
   role.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
   clusterrole.rbac.authorization.k8s.io/ingress-nginx unchanged
   clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
   rolebinding.rbac.authorization.k8s.io/ingress-nginx unchanged
   rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
   clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx unchanged
   clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
   configmap/ingress-nginx-controller configured
   service/ingress-nginx-controller configured
   service/ingress-nginx-controller-admission unchanged
   deployment.apps/ingress-nginx-controller configured
   job.batch/ingress-nginx-admission-create unchanged
   job.batch/ingress-nginx-admission-patch unchanged
   ingressclass.networking.k8s.io/nginx unchanged
   validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission unchanged
   ```

1. 以下コマンドでhttpsで通信できることを確認してください。（`httpではない`）

   ```bash
   curl https://test-1.k8s.practice.local
   curl https://test-1.k8s.practice.local
   ```

   【回答例】

   ```bash
   # 実行結果
   $ curl https://test-1.k8s.practice.local
   nginx-1-f57b57756-7zlfd

   $ curl https://test-2.k8s.practice.local
   nginx-2-68d8c9ff8-8b96q
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f Ingress-nginx-1.yaml
   deployment.apps "nginx-1" deleted
   service "nginx-1-svc" deleted
   ingress.networking.k8s.io "nginx-1-ingress" deleted

   $ kubectl delete -f Ingress-nginx-2.yaml
   deployment.apps "nginx-2" deleted
   service "nginx-2-svc" deleted
   ingress.networking.k8s.io "nginx-2-ingress" deleted

   $ kubectl delete -f Ingress-deploy.yaml
   namespace/ingress-nginx deleted
   serviceaccount/ingress-nginx deleted
   serviceaccount/ingress-nginx-admission deleted
   role.rbac.authorization.k8s.io/ingress-nginx deleted
   role.rbac.authorization.k8s.io/ingress-nginx-admission deleted
   clusterrole.rbac.authorization.k8s.io/ingress-nginx deleted
   clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission deleted
   rolebinding.rbac.authorization.k8s.io/ingress-nginx deleted
   rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission deleted
   clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx deleted
   clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission deleted
   configmap/ingress-nginx-controller deleted
   service/ingress-nginx-controller deleted
   service/ingress-nginx-controller-admission deleted
   deployment.apps/ingress-nginx-controller deleted
   job.batch/ingress-nginx-admission-create deleted
   job.batch/ingress-nginx-admission-patch deleted
   ingressclass.networking.k8s.io/nginx deleted
   validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission deleted
   ```

[1]:https://kubernetes.io/docs/concepts/services-networking/ingress/
