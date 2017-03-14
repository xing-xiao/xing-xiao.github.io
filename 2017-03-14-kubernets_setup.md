# kubernetes集群安装脚本


### 系统版本
centos 7
### 执行脚本

在master上执行脚本，其中watch的地方，等待pod状态全部变为running后切出。记住`kubeadm init`操作生成的token。我们这里使用weave网络，当然你也可以使用calico、Romana。

```shell
#!/bin/bash
yum update -y
yum install -y socat     
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
curl -sSL http://k8s.oss-cn-shanghai.aliyuncs.com/kube/rpm/kubectl-1.5.1.x86_64.rpm -o kubectl-1.5.1.x86_64.rpm 
curl -sSL http://k8s.oss-cn-shanghai.aliyuncs.com/kube/rpm/kubelet-1.5.1.x86_64.rpm -o kubelet-1.5.1.x86_64.rpm 
curl -sSL http://k8s.oss-cn-shanghai.aliyuncs.com/kube/rpm/kubernetes-cni-0.3.0.1-1.07a8a2.x86_64.rpm -o kubernetes-cni-0.3.0.1-1.07a8a2.x86_64.rpm 
curl -sSL http://k8s.oss-cn-shanghai.aliyuncs.com/kube/rpm/kubeadm-1.6.0-0.alpha.0.2074.a092d8e0f95f52.x86_64.rpm -o kubeadm-1.6.0-0.alpha.0.2074.a092d8e0f95f52.x86_64.rpm 
rpm -ivh kubectl-1.5.1.x86_64.rpm kubelet-1.5.1.x86_64.rpm kubernetes-cni-0.3.0.1-1.07a8a2.x86_64.rpm  kubeadm-1.6.0-0.alpha.0.2074.a092d8e0f95f52.x86_64.rpm
echo "[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--kubeconfig=/etc/kubernetes/kubelet.conf --require-kubeconfig=true"
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0"
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_DNS_ARGS=--cluster-dns=10.96.0.10 --cluster-domain=cluster.local"
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_EXTRA_ARGS" > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
setenforce 0
systemctl stop firewalld && systemctl disable firewalld
systemctl enable docker && systemctl start docker && systemctl status docker
systemctl enable kubelet && systemctl start kubelet && systemctl status kubelet
export KUBE_REPO_PREFIX=registry.cn-hangzhou.aliyuncs.com/google-containers \
         KUBE_HYPERKUBE_IMAGE=registry.cn-hangzhou.aliyuncs.com/google-containers/hyperkube-amd64:v1.5.1 \
         KUBE_DISCOVERY_IMAGE=registry.cn-hangzhou.aliyuncs.com/google-containers/kube-discovery-amd64:1.0 \
         KUBE_ETCD_IMAGE=registry.cn-hangzhou.aliyuncs.com/google-containers/etcd-amd64:3.0.4
#kubeadm init --pod-network-cidr="10.24.0.0/16"
kubeadm init --skip-preflight-checks
kubectl taint nodes --all dedicated-
kubectl apply -f http://k8s.oss-cn-shanghai.aliyuncs.com/kube/weave-kube-1.7.2
watch -n 5 kubectl get pod --namespace=kube-system
kubectl apply -f http://k8s.oss-cn-shanghai.aliyuncs.com/kube/kubernetes-dashboard1.5.0.yaml
kubectl --namespace=kube-system describe svc kubernetes-dashboard
watch -n 5 kubectl get pod --namespace=kube-system
```

在minion上执行脚本

```shell
#!/bin/bash
yum update -y
yum install -y tmux socat
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
curl -sSL http://k8s.oss-cn-shanghai.aliyuncs.com/kube/rpm/kubectl-1.5.1.x86_64.rpm -o kubectl-1.5.1.x86_64.rpm 
curl -sSL http://k8s.oss-cn-shanghai.aliyuncs.com/kube/rpm/kubelet-1.5.1.x86_64.rpm -o kubelet-1.5.1.x86_64.rpm 
curl -sSL http://k8s.oss-cn-shanghai.aliyuncs.com/kube/rpm/kubernetes-cni-0.3.0.1-1.07a8a2.x86_64.rpm -o kubernetes-cni-0.3.0.1-1.07a8a2.x86_64.rpm 
curl -sSL http://k8s.oss-cn-shanghai.aliyuncs.com/kube/rpm/kubeadm-1.6.0-0.alpha.0.2074.a092d8e0f95f52.x86_64.rpm -o kubeadm-1.6.0-0.alpha.0.2074.a092d8e0f95f52.x86_64.rpm 
rpm -ivh kubectl-1.5.1.x86_64.rpm kubelet-1.5.1.x86_64.rpm kubernetes-cni-0.3.0.1-1.07a8a2.x86_64.rpm  kubeadm-1.6.0-0.alpha.0.2074.a092d8e0f95f52.x86_64.rpm
echo "[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--kubeconfig=/etc/kubernetes/kubelet.conf --require-kubeconfig=true"
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0"
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_DNS_ARGS=--cluster-dns=10.96.0.10 --cluster-domain=cluster.local"
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_EXTRA_ARGS" > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
setenforce 0
systemctl stop firewalld && systemctl disable firewalld
systemctl enable docker && systemctl start docker && systemctl status docker
systemctl enable kubelet && systemctl start kubelet && systemctl status kubelet
kubeadm join --token=${your token here} ${your k8s master ip} --skip-preflight-checks
```