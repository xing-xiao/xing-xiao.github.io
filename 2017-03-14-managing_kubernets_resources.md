# 管理kubernetes资源

## 资源文件

我们可以通过yaml或者json格式文件创建kubernetes资源。一个简单的例子如下所示，在yaml文件中可以在一个文件中写入多个资源，使用`---`分隔。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-svc
  labels:
    app: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: nginx
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
在这个例子中可以创建一个service和一个deployment，资源的创建是按照文件中资源的顺序来的，所以我们最好把service放在最开始，然后再布置deployment。


## 资源操作

### 资源创建

资源的创建使用`kubectl create -f`命令，可以有以下几种用法：

- `kubectl create -f docs/user-guide/nginx-app.yaml`，使用文件名
- `kubectl create -f docs/user-guide/nginx/`，使用文件路径
- `kubectl create -f https://raw.githubusercontent.com/kubernetes/kubernetes/master/docs/user-guide/nginx-deployment.yaml`，使用URL

使用`-o`参数可以打印资源创建内容

```shell
$ kubectl get $(kubectl create -f docs/user-guide/nginx/ -o name | grep service)
NAME           CLUSTER-IP   EXTERNAL-IP   PORT(S)      AGE
my-nginx-svc   10.0.0.208                 80/TCP       0s
```
使用`--recursive`参数可以创建目录下所有子目录中资源文件

### 资源删除

同样可以对资源文件使用`kubectl delete -f`删除资源，如：

- `kubectl delete -f docs/user-guide/nginx/`

删除操作中使用`-l`参数可以针对特定label的资源进行删除，

- `kubectl delete deployment,services -l app=nginx`

### 应用副本伸缩

可以使用`kubectl scale`来重新配置应用的副本数量，如

- `kubectl scale deployment/my-nginx --replicas=1`将副本数量重置为1

也可以用`kubectl autoscale`来给副本数量设定一个阈值，如

- `kubectl autoscale deployment/my-nginx --min=1 --max=3`将副本数量设置为1到3

## 资源升级

### kubectl apply

`kubectl apply`可以在不影响应用运行的时候对资源进行升级，他仅仅对配置文件中更改的资源进行从新配置，不会影响配置文件中没有更改的资源。当资源不存在时则创建资源。使用`kubectl apply`，要求资源使用`kubectl apply`或者`kubectl create --save-config`

`kubectl apply`对现有资源使用 2-way diff，资源创建后，使用命令对资源进行了更改，然后再使用`kubectl apply`时，原本的更改内容并不会被保留。而`kubectl edit`和`kubectl replace`则会保留原来的更改。

- `kubectl apply -f docs/user-guide/nginx/nginx-deployment.yaml`

### kubectl edit

用法同上，如

- `kubectl edit deployment/my-nginx`

相当于如下操作

- `kubectl get deployment my-nginx -o yaml > /tmp/nginx.yaml`
- edit the file
- `kubectl apply -f /tmp/nginx.yaml`

### kubectl rolling-update

`rolling-update`可以对于多个副本的pod、rc、depolyment进行逐个升级，以避免因升级而导致的服务暂停。用法可以参考官方给出的例子，如下：

```
  # Update pods of frontend-v1 using new replication controller data in frontend-v2.json.
  kubectl rolling-update frontend-v1 -f frontend-v2.json
  
  # Update pods of frontend-v1 using JSON data passed into stdin.
  cat frontend-v2.json | kubectl rolling-update frontend-v1 -f -
  
  # Update the pods of frontend-v1 to frontend-v2 by just changing the image, and switching the
  # name of the replication controller.
  kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2
  
  # Update the pods of frontend by just changing the image, and keeping the old name.
  kubectl rolling-update frontend --image=image:v2
  
  # Abort and reverse an existing rollout in progress (from frontend-v1 to frontend-v2).
  kubectl rolling-update frontend-v1 frontend-v2 --rollback
```

对于deployment，rolling-update需要在deployment的yaml/json文件中spec段设置：

```
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```