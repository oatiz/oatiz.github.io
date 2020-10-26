+++
title = "k8s部署笔记"
date = 2018-12-10

[taxonomies]
tags = ["excerpts", "kubernetes"]
categories = ["Kubernetes"]
+++

## centOS

1. 确认centos 7 内核为3.10以上

<!-- more -->

2. 关闭swap (使用命令 swapoff -a 进行暂时关闭，如果永久关闭，自己查方法吧)

3. 设置selinux 为permissive 状态
    执行命令:
        setenforce 0
        sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

4. 安装kubeadm
    执行下面命令 设置yum源

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
执行命令安装(同时会将其他组件一并安装)
yum install kubeadm  

5. 执行初始化 

>kubeadm init  --image-repository nidynie

—image-repository  指明拿基础镜像的域名 默认是k8s.gcr.io 但是国内无法访问  nidynie是我从docker hub上做个一个mirror 对应 k8s.gcr.io 的v1.13.1 版本，所有可以直接使用

成功之后，系统会提示执行以下命令，逐条执行即可
```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## ubuntu

[部署高可用Kubernetes集群](https://qingmu.io/2018/12/20/Deploy-a-highly-available-cluster-with-kubeadm/#%E5%8D%87%E7%BA%A7%E5%86%85%E6%A0%B8)
