# 回答例

1. Nginx Ingress Controllerをデプロイしてください。なお、[Nginx Ingress Controllerの公式][1]を参考にしてください。次章でTLS終端を行いますので、使用するmanifestに注意してください。

   > :information_source:  
   > Nginx Ingress は Ver.1.6.4を使用しています。
   > 今後のアップデートによって、使用するmanifestや手順が変更になる可能性があります。
   > その際は公式の最新マニュアルを参照して、適宜読み替えてください。

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: Namespace
   metadata:
     labels:
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
     name: ingress-nginx
   ---
   apiVersion: v1
   automountServiceAccountToken: true
   kind: ServiceAccount
   metadata:
     labels:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx
     namespace: ingress-nginx
   ---
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     labels:
       app.kubernetes.io/component: admission-webhook
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-admission
     namespace: ingress-nginx
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     labels:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx
     namespace: ingress-nginx
   rules:
     - apiGroups:
         - ""
       resources:
         - namespaces
       verbs:
         - get
     - apiGroups:
         - ""
       resources:
         - configmaps
         - pods
         - secrets
         - endpoints
       verbs:
         - get
         - list
         - watch
     - apiGroups:
         - ""
       resources:
         - services
       verbs:
         - get
         - list
         - watch
     - apiGroups:
         - networking.k8s.io
       resources:
         - ingresses
       verbs:
         - get
         - list
         - watch
     - apiGroups:
         - networking.k8s.io
       resources:
         - ingresses/status
       verbs:
         - update
     - apiGroups:
         - networking.k8s.io
       resources:
         - ingressclasses
       verbs:
         - get
         - list
         - watch
     - apiGroups:
         - coordination.k8s.io
       resourceNames:
         - ingress-nginx-leader
       resources:
         - leases
       verbs:
         - get
         - update
     - apiGroups:
         - coordination.k8s.io
       resources:
         - leases
       verbs:
         - create
     - apiGroups:
         - ""
       resources:
         - events
       verbs:
         - create
         - patch
     - apiGroups:
         - discovery.k8s.io
       resources:
         - endpointslices
       verbs:
         - list
         - watch
         - get
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     labels:
       app.kubernetes.io/component: admission-webhook
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-admission
     namespace: ingress-nginx
   rules:
     - apiGroups:
         - ""
       resources:
         - secrets
       verbs:
         - get
         - create
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     labels:
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx
   rules:
     - apiGroups:
         - ""
       resources:
         - configmaps
         - endpoints
         - nodes
         - pods
         - secrets
         - namespaces
       verbs:
         - list
         - watch
     - apiGroups:
         - coordination.k8s.io
       resources:
         - leases
       verbs:
         - list
         - watch
     - apiGroups:
         - ""
       resources:
         - nodes
       verbs:
         - get
     - apiGroups:
         - ""
       resources:
         - services
       verbs:
         - get
         - list
         - watch
     - apiGroups:
         - networking.k8s.io
       resources:
         - ingresses
       verbs:
         - get
         - list
         - watch
     - apiGroups:
         - ""
       resources:
         - events
       verbs:
         - create
         - patch
     - apiGroups:
         - networking.k8s.io
       resources:
         - ingresses/status
       verbs:
         - update
     - apiGroups:
         - networking.k8s.io
       resources:
         - ingressclasses
       verbs:
         - get
         - list
         - watch
     - apiGroups:
         - discovery.k8s.io
       resources:
         - endpointslices
       verbs:
         - list
         - watch
         - get
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     labels:
       app.kubernetes.io/component: admission-webhook
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-admission
   rules:
     - apiGroups:
         - admissionregistration.k8s.io
       resources:
         - validatingwebhookconfigurations
       verbs:
         - get
         - update
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     labels:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx
     namespace: ingress-nginx
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: ingress-nginx
   subjects:
     - kind: ServiceAccount
       name: ingress-nginx
       namespace: ingress-nginx
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     labels:
       app.kubernetes.io/component: admission-webhook
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-admission
     namespace: ingress-nginx
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: ingress-nginx-admission
   subjects:
     - kind: ServiceAccount
       name: ingress-nginx-admission
       namespace: ingress-nginx
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
     labels:
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: ingress-nginx
   subjects:
     - kind: ServiceAccount
       name: ingress-nginx
       namespace: ingress-nginx
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
     labels:
       app.kubernetes.io/component: admission-webhook
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-admission
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: ingress-nginx-admission
   subjects:
     - kind: ServiceAccount
       name: ingress-nginx-admission
       namespace: ingress-nginx
   ---
   apiVersion: v1
   data:
     allow-snippet-annotations: "true"
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
       service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
       service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
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
         targetPort: http
       - appProtocol: https
         name: https
         port: 443
         protocol: TCP
         targetPort: https
     selector:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
     type: LoadBalancer
   ---
   apiVersion: v1
   kind: Service
   metadata:
     labels:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-controller-admission
     namespace: ingress-nginx
   spec:
     ports:
       - appProtocol: https
         name: https-webhook
         port: 443
         targetPort: webhook
     selector:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
     type: ClusterIP
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
               - containerPort: 443
                 name: https
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
   apiVersion: batch/v1
   kind: Job
   metadata:
     labels:
       app.kubernetes.io/component: admission-webhook
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-admission-create
     namespace: ingress-nginx
   spec:
     template:
       metadata:
         labels:
           app.kubernetes.io/component: admission-webhook
           app.kubernetes.io/instance: ingress-nginx
           app.kubernetes.io/name: ingress-nginx
           app.kubernetes.io/part-of: ingress-nginx
           app.kubernetes.io/version: 1.6.4
         name: ingress-nginx-admission-create
       spec:
         containers:
           - args:
               - create
               - --host=ingress-nginx-controller-admission,ingress-nginx-controller-admission.$(POD_NAMESPACE).svc
               - --namespace=$(POD_NAMESPACE)
               - --secret-name=ingress-nginx-admission
             env:
               - name: POD_NAMESPACE
                 valueFrom:
                   fieldRef:
                     fieldPath: metadata.namespace
             image: registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20220916-gd32f8c343@sha256:39c5b2e3310dc4264d638ad28d9d1d96c4cbb2b2dcfb52368fe4e3c63f61e10f
             imagePullPolicy: IfNotPresent
             name: create
             securityContext:
               allowPrivilegeEscalation: false
         nodeSelector:
           kubernetes.io/os: linux
         restartPolicy: OnFailure
         securityContext:
           fsGroup: 2000
           runAsNonRoot: true
           runAsUser: 2000
         serviceAccountName: ingress-nginx-admission
   ---
   apiVersion: batch/v1
   kind: Job
   metadata:
     labels:
       app.kubernetes.io/component: admission-webhook
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-admission-patch
     namespace: ingress-nginx
   spec:
     template:
       metadata:
         labels:
           app.kubernetes.io/component: admission-webhook
           app.kubernetes.io/instance: ingress-nginx
           app.kubernetes.io/name: ingress-nginx
           app.kubernetes.io/part-of: ingress-nginx
           app.kubernetes.io/version: 1.6.4
         name: ingress-nginx-admission-patch
       spec:
         containers:
           - args:
               - patch
               - --webhook-name=ingress-nginx-admission
               - --namespace=$(POD_NAMESPACE)
               - --patch-mutating=false
               - --secret-name=ingress-nginx-admission
               - --patch-failure-policy=Fail
             env:
               - name: POD_NAMESPACE
                 valueFrom:
                   fieldRef:
                     fieldPath: metadata.namespace
             image: registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20220916-gd32f8c343@sha256:39c5b2e3310dc4264d638ad28d9d1d96c4cbb2b2dcfb52368fe4e3c63f61e10f
             imagePullPolicy: IfNotPresent
             name: patch
             securityContext:
               allowPrivilegeEscalation: false
         nodeSelector:
           kubernetes.io/os: linux
         restartPolicy: OnFailure
         securityContext:
           fsGroup: 2000
           runAsNonRoot: true
           runAsUser: 2000
         serviceAccountName: ingress-nginx-admission
   ---
   apiVersion: networking.k8s.io/v1
   kind: IngressClass
   metadata:
     labels:
       app.kubernetes.io/component: controller
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: nginx
   spec:
     controller: k8s.io/ingress-nginx
   ---
   apiVersion: admissionregistration.k8s.io/v1
   kind: ValidatingWebhookConfiguration
   metadata:
     labels:
       app.kubernetes.io/component: admission-webhook
       app.kubernetes.io/instance: ingress-nginx
       app.kubernetes.io/name: ingress-nginx
       app.kubernetes.io/part-of: ingress-nginx
       app.kubernetes.io/version: 1.6.4
     name: ingress-nginx-admission
   webhooks:
     - admissionReviewVersions:
         - v1
       clientConfig:
         service:
           name: ingress-nginx-controller-admission
           namespace: ingress-nginx
           path: /networking/v1/ingresses
       failurePolicy: Fail
       matchPolicy: Equivalent
       name: validate.nginx.ingress.kubernetes.io
       rules:
         - apiGroups:
             - networking.k8s.io
           apiVersions:
             - v1
           operations:
             - CREATE
             - UPDATE
           resources:
             - ingresses
       sideEffects: None
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f Ingress-deploy.yaml
   namespace/ingress-nginx created
   serviceaccount/ingress-nginx created
   serviceaccount/ingress-nginx-admission created
   role.rbac.authorization.k8s.io/ingress-nginx created
   role.rbac.authorization.k8s.io/ingress-nginx-admission created
   clusterrole.rbac.authorization.k8s.io/ingress-nginx created
   clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
   rolebinding.rbac.authorization.k8s.io/ingress-nginx created
   rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
   clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
   clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
   configmap/ingress-nginx-controller created
   service/ingress-nginx-controller created
   service/ingress-nginx-controller-admission created
   deployment.apps/ingress-nginx-controller created
   job.batch/ingress-nginx-admission-create created
   job.batch/ingress-nginx-admission-patch created
   ingressclass.networking.k8s.io/nginx created
   validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
   ```

1. AWSマネジメントコンソール（コマンドでも可）でNginx IngressのELBが作成されていることを確認してください。ELBのタグを見ればNginx Ingressのものか判断できます。

   【回答例】

   ![Nginx-Ingress](../images/Nginx-Ingress.png)

[1]:https://kubernetes.github.io/ingress-nginx/deploy/#tls-termination-in-aws-load-balancer-nlb