# kuberntest pod

## overview

pod是kubernetes中最基本的building block，一个容器就是一个kubernetes实例。

pod由一个或者多个container组成。pod中所有的container都共用一个网络空间，可以通过localhost在一个pod中不同的container之间访问。pod同时可以指定不同container对存储、CPU、内存等资源的使用。

## 创建pod

### pod配置文件

我们可以通过配置文件创建pod，以一个例子来解释基本的文件结构，如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "1"
    volumeMounts:
    - name: storage
      mountPath: /data/redis
  volumes:
  - name: storage
    emptyDir: {}
  nodeSelector:
    kubernetes.io/hostname: k8s-minion-2
```

- `apiVersion: v1`							使用的api版本
- `kind: pod` 								指定创建的是pod资源
- `metadata.name: demon` 					pod名称
- `metadata.labels:<key>:<value>`			pod的label
- `spec:`									操作，包括container、volume等
- `spec.containers:`						containers配置
- `spec.containers.name:` 					container名称
- `spec.containers.image:` 					container使用的image
- `spec.containers.imagePullPolicy:`		值为`IfNotPresent`时，如果node上img已经存在则不会再从repository中拉取img，否则会从repository中拉取最新img
- `spec.containers.ports:<containerPort>:$(port)`					container暴露的port
- `spec.containers.env:` 					container中环境变量
- `spec.containers.env.name:`				container中环境变量名称
- `spec.containers.env.value:` 				container中环境变量值
- `spec.containers.command:` 				container运行的命令
- `spec.containers.args:`					container运行参数
*在这里，如果没有设置command 和args，则使用原有img中`ENTRYPOINT`和`CMD`；如果仅设置args不设置command，则会使用原有img中`ENTRYPOINT`和这里设置的参数；如果设置了command，这里的command会覆盖img中的`ENTRYPOINT`和`CMD`。*
- `spec.containers.resources:` 				container使用的资源限制
- `spec.containers.resources.requests:<memory>:$(value)/<cpu>:$(value)`		container需要分配的最小资源，仅当node上可以提供requests所需的资源时，才会将pod调度到该node
- `spec.containers.resources.limits:<memory>:$(value)/<cpu>:$(value)` 		会分配给container的最大资源
- `spec.containers.volumeMounts:<name>:$(value) <mountPath>:$(value)`						容器向pod挂载的路径
- `spec.volumes:`							pod提供的挂载
- `spec.nodeSelector:`						可以通过该选项指定pod运行的node
```
emptyDir:{}			空路径，不同container挂载路径均映射到emptyDir，共同对其中的文件进行操作。容器crash不会影响emptyDir中文件。
hostPath:path: /	挂载一个node上的路径到你的pod，但是宿主机创建的文件，除了privileged container，其它container没有写权限。
```

运行起来试试看

`kubectl get pods -l purpose=demonstrate-envars`

`kubectl exec -it envar-demo -- printenv`

## 运行多个container

```
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:

  restartPolicy: Never

  volumes:
  - name: shared-data
    emptyDir: {}

  containers:

  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

  - name: debian-container
    image: debian
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]
```

一个pod可以运行多个container，每个container可以通过`localhost`访问其他container开放的端口，也可以通过共享的volume来进行通信。

## init container

`init container`是在pod中正式container之前运行的contaneir，它可以用来：

1. 预处理数据，如`git clone, wget`或一些运算结果，将处理结果放到volume中以供正式container使用。
- 阻塞正式container等待其他运行环境准备完毕。例如一个前端程序依赖后端数据库，可以使用`init container`来阻塞前端contianer启动，等待后端程序启动后再退出`init container`再启动前端程序。

一个例子如下，阻塞container运行，等待myservice和mydb启动：
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
  annotations:
    pod.beta.kubernetes.io/init-containers: '[
        {
            "name": "init-myservice",
            "image": "busybox",
            "command": ["sh", "-c", "until nslookup myservice; do echo waiting for myservice; sleep 2; done;"]
        },
        {
            "name": "init-mydb",
            "image": "busybox",
            "command": ["sh", "-c", "until nslookup mydb; do echo waiting for mydb; sleep 2; done;"]
        }
    ]'
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```

也可以使用`for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; exit 1`等待myservice启动。

## DownwardApi
待补充
