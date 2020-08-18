---
layout: post
title: "k8s笔记：创建k8s的admin权限token并使用"
subtitle: "使用k8s创建admin权限token"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - k8s
---
# k8s admin token

## 1. 创建admin账户

### 1.1 yaml 文件

```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
```

### 1.2 创建admin账户

```
kubectl apply -f admin.yaml
```



## 2. 查看token

### 2.1 获取token的sh脚本

```
kubectl describe secret $(kubectl get secret -n kube-system | grep admin | awk '{ print $1}') -n kube-system
```

### 2.2 token字段即admin权限对应的token

```
Name:         admin-token-6g67p
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin
              kubernetes.io/service-account.uid: b2231259-e11b-11ea-990a-68b599b765a0

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi02ZzY3cCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImIyMjMxMjU5LWUxMWItMTFlYS05OTBhLTY4YjU5OWI3NjVhMCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.PdMLMDNHhcbhTIubmXVLzTjSbhegYKIJwLfBFOBrPwDN85FDwIAQyeE8j2VjXF69DQRgQVxGXq9tAk5fKRncyuPIedzujNrW2ZGJcHI9EQHM21LecCGTdXnadKC5GnLB4ny5K3eXReKS6dtzBbT5qoZ6x36c8O0ajILjPdH8J4BbkjHfUGewo0HuVCrTIFkPSyDZrZ67aC-Zv7Vvm3USZKL3yFt_IpZAqt_ToJ0RzY40Cg4ox9b1OyPWa5Bn9ED7Hu4H42dLMeSmAeYfsjvqYCLqX62PJ1gPyeE5mvT3YEkFgjW04w8DrdholrhKR-3SzLSbf6GUIphF4QZP2tbBgg
```



## 3. 使用token设置config文件

```
$ cat ~/.kube/config
piVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: <server地址>
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: webkubectl-user
  name: kubernetes
current-context: kubernetes
kind: Config
preferences: {}
users:
- name: webkubectl-user
  user:
    token: <上述token>
```

