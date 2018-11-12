---
layout: post
title:  "Kubernetesことはじめ"
date:   2018-11-10 12:00:00
category: kubernetes
tags:
    - docker
    - Windows
---

## kubernetes-cli インストールします

```bash
$ choco install kubernetes-cli
$ kubectl version
```

[Install and Set Up kubectl - Kubernetes](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-chocolatey-on-windows)


## Dashboard をデプロイします

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

[kubernetes/dashboard: General-purpose web UI for Kubernetes clusters](https://github.com/kubernetes/dashboard)


## ダッシュボードへのプロキシサーバーを立ち上げます

```bash
$ kubectl proxy
```

[概要 - Kubernetes Dashboard](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default)


## Pod

### サンプル pod.yaml を実行してみます

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: example-app
  labels:
    app: example-app
spec:
  containers:
  - name: example-app
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

上記 Pod をローカル **Kubernetes クラスタ** に反映してみます。

```bash
$ kubectl apply -f pod.yaml
```

コンテナが実行状態になったか確認してみます。

```bash
$ kubectl get pod
NAME          READY     STATUS    RESTARTS   AGE
example-app   1/1       Running   0          1m
```

**Running** になっていますね！

Pod の詳細も見てみましょう。

```bash
$ kubectl describe pods/example-app
Name:         example-app
Namespace:    default
Node:         docker-for-desktop/192.168.65.3
Start Time:   Wed, 31 Oct 2018 16:03:20 +0900
Labels:       app=example-app
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"example-app"},"name":"example-app","namespace":"default"},"spec":{"contai...
Status:       Running
IP:           10.1.0.65
Containers:
  example-app:
    Container ID:   docker://5c6c97e5529785740c06abea2e407d569938f5250b4b12127436b7eb82a0f0cd
    Image:          nginx:1.7.9
    Image ID:       docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 31 Oct 2018 16:03:32 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8jjjx (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-8jjjx:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-8jjjx
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From                         Message
  ----    ------                 ----  ----                         -------
  Normal  Scheduled              3m    default-scheduler            Successfully assigned example-app to docker-for-desktop
  Normal  SuccessfulMountVolume  3m    kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-8jjjx"
  Normal  Pulling                3m    kubelet, docker-for-desktop  pulling image "nginx:1.7.9"
  Normal  Pulled                 3m    kubelet, docker-for-desktop  Successfully pulled image "nginx:1.7.9"
  Normal  Created                3m    kubelet, docker-for-desktop  Created container
  Normal  Started                3m    kubelet, docker-for-desktop  Started container
```

### ログを標準出力します

```bash
$ kubectl logs -f example-app
```

### オブジェクトを削除します

```bash
$ kubectl delete pods/example-app
```

直後に get pod すると、 **Terminating** 状態になりました。

```bash
$ kubectl get pod
NAME          READY     STATUS        RESTARTS   AGE
example-app   0/1       Terminating   0          9m
```

しばらくすると、

```bash
$ kubectl get pod
No resources found.
```

削除されました！


## Deployment

### サンプル deployment.yaml を実行してみます

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: example-app-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-app-deploy
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

    「ラベルが **app: example-app** と一致するポッドが、レプリカ数 **3** で展開される」

Kubernetes API サーバに Deployment マニフェストを送信します。

```bash
$ kubectl apply -f deployment.yaml
```

**replicas: 3** のとおり、 3 つ作られていそうです。

```bash
$ kubectl get deployments
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
example-app-deploy   3         3         3            3           1m
```

Pod も見てみましょう。

**example-app-deploy-xxxx** というのが 3 つ作られていますね！

```bash
$ kubectl get pods
NAME                                  READY     STATUS    RESTARTS   AGE
example-app                           1/1       Running   0          12m
example-app-deploy-576bfb99df-8vtp7   1/1       Running   0          1m
example-app-deploy-576bfb99df-z7twc   1/1       Running   0          1m
example-app-deploy-576bfb99df-zqggg   1/1       Running   0          1m
```

以下のように情報をまとめて見ることもできます。

このコマンドは今後すごく使いそうです。

```bash
$ kubectl get pod,replicaset,deployment
NAME                                      READY     STATUS    RESTARTS   AGE
pod/example-app                           1/1       Running   0          16m
pod/example-app-deploy-576bfb99df-8vtp7   1/1       Running   0          5m
pod/example-app-deploy-576bfb99df-z7twc   1/1       Running   0          5m
pod/example-app-deploy-576bfb99df-zqggg   1/1       Running   0          5m

NAME                                                  DESIRED   CURRENT   READY     AGE
replicaset.extensions/example-app-deploy-576bfb99df   3         3         3         5m

NAME                                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/example-app-deploy   3         3         3            3           5m
```

## Service

### サンプル service.yaml を実行してみます

デフォルトの **ClusterIP** ベースになります。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: example-service
spec:
  selector:
    app: example-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```bash
$ kubectl apply -f service.yaml
```

```bash
$ kubectl get svc
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
example-service   ClusterIP   10.102.217.235   <none>        80/TCP    35s
```

**svc** は **services** のエイリアスです。

```bash
$ kubectl get services
```

なので、これでも動きます。

### Service にアクセスしてみます

Service には Kubernetes クラスタの内部からしかアクセスできません。

なので、プロキシでサービスにアクセスしてみましょう。

```bash
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

[Welcome to nginx!](http://127.0.0.1:8001/api/v1/namespaces/default/services/example-service/proxy/)

nginx の画面が出ればアクセス成功です！

### サンプル nodeport-service.yaml を実行してみます

上記の service.yaml に **type: NodePort** を追加しただけです。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: example-service
spec:
  type: NodePort
  selector:
    app: example-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```bash
$ kubectl apply -f nodeport-service.yaml
```

```bash
$ kubectl get service
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
example-service   NodePort    10.102.217.235   <none>        80:32342/TCP   18m```
```

PORT に **80:32342** と書かれており、ホスト側と Service がポートフォワードされています。

[Welcome to nginx!](http://localhost:32342/)


## 1 つのファイルにまとめて記述してみます

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: example-app-k8s
spec:
  replicas: 1
  selector:
    matchLabels:
      app: example-app-k8s
  template:
    metadata:
      labels:
        app: example-app-k8s
    spec:
      containers:
      - name: example-app-k8s
        image: "example-app:latest"
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: example-app-k8s
spec:
  type: NodePort
  selector:
    app: example-app-k8s
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
```

ローカルの image を使う場合は、 **imagePullPolicy: IfNotPresent** を指定します。

## 読んだ記事

[Kubernetesを始めよう！ - DMM inside](https://inside.dmm.com/entry/2018/04/13/hello-kubernetes)

[Kubernetes の Pod / ReplicaSet / Deployment について、ようやく整理できた （入門k8s 読書メモ） - えいのうにっき](https://blog.a-know.me/entry/2018/08/14/185324)

[Docker for Mac [Edge] で Kubernetes を試してみる - Qiita](https://qiita.com/mediba-moritake/items/6cce164c3bda317c32b8)
