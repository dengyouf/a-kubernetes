# 安装Kubernetes集群

- 配置kubernetes仓库

```
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt-get update

```

- 安装kubeadm

```
apt install -y kubeadm=1.23.5-00 kubelet=1.23.5-00 kubectl=1.23.5-00

systemctl  enable kubelet
```

- 拉取镜像

```
kubeadm config images pull --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers --kubernetes-version=1.23.5
```

## 初始化集群

```
# 所有节点需要解析 vip.linux.io
 echo "192.168.122.111 k8s-vip vip.linux.io" >> /etc/hosts

kubeadm init --kubernetes-version=v1.23.5 \
    --control-plane-endpoint=vip.linux.io \
    --apiserver-advertise-address=0.0.0.0 \
    --pod-network-cidr=10.244.0.0/16   \
    --service-cidr=10.96.0.0/12 \
    --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
    --ignore-preflight-errors=Swap | tee kubeadm-init.log
```

## 配置kubectl命令

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Join工作节点

```
kubeadm join vip.linux.io:6443 --token lz54lr.35cyylrtl3gspe1c \
        --discovery-token-ca-cert-hash sha256:c7aab2ac8f872667eb2bfc3582cc707508e2af6c7e56188bc1022a827c7b88f5
```

## 安装CNI

- flannel 
```
wget  https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

## 验证集群

```
~# kubectl  create deployment myapp --image=ikubernetes/myapp:v1 --replicas=3
 kubectl  expose deployment myapp --port=80 --target-port=80
```

```
~# kubectl  get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE    IP           NODE     NOMINATED NODE   READINESS GATES
myapp-9cbc4cf76-7cgwt   1/1     Running   0          103s   10.244.1.4   k8s-w1   <none>           <none>
myapp-9cbc4cf76-qzghs   1/1     Running   0          103s   10.244.2.2   k8s-w2   <none>           <none>
myapp-9cbc4cf76-xmgxt   1/1     Running   0          103s   10.244.2.3   k8s-w2   <none>           <none>
root@k8s-m1:~# kubectl  get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   15m
myapp        ClusterIP   10.97.183.10   <none>        80/TCP    53s
root@k8s-m1:~# for i in `seq 4`;do curl 10.97.183.10/hostname.html;done
myapp-9cbc4cf76-7cgwt
myapp-9cbc4cf76-7cgwt
myapp-9cbc4cf76-qzghs
myapp-9cbc4cf76-7cgwt
```