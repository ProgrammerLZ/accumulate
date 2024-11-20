
切记，切换国内镜像。

```shell
# 切换国内镜像
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
echo deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main >/etc/apt/sources.list.d/kubernetes.list

# 下载三驾马车
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

