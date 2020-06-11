

```
---
layout: post
title: "k8s笔记：serverless-openfaas体验"
subtitle: "centos7下部署和使用openfaas"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - k8s
  - helm
---
```

## 1. 部署

> 在centos7.2上部署使用

1. 使用minikube部署k8s集群，已经有k8s集群的略过。

2. 安装openfaas-cli

   ```
   curl -sL https://cli.openfaas.com | sh
   ```

3. 使用k8s的yaml文件安装openfaas

   1. 创建namespace

      ```
      git clone https://github.com/openfaas/faas-netes
      cd faas-netes
      kubectl apply -f namespaces.yml
      ```

   2. 创建admin的密码

      ```
      $ cat passwd.sh
          # PASSWORD=$(head -c 12 /dev/urandom | sha1sum |cut -d' ' -f1)
          PASSWORD=admin123
          echo $PASSWORD > passwd
          kubectl -n openfaas create secret generic basic-auth \
          --from-literal=basic-auth-user=admin \
          --from-literal=basic-auth-password="$PASSWORD"
      $ sh ./passwd.sh
      ```

   3. 部署openfaas

      ```
      $ kubectl apply -f ./yaml
      $ kubectl get pods -n openfaas --watch 
      # 等待全部running
      ```

   4. 设置环境变量OPENFAAS_URL

      ```
      $ export OPENFAAS_URL=127.0.0.1:31112
      备注： pod的8080端口，如果在集群外面，可以通过创建proxy来访问，kubectl port-forward svc/gateway -n openfaas 31112:8080 &
      ```

   5. 登录 openfaas

      ```
      $ export OPENFAAS_URL=http://127.0.0.1:31112
      $ cat passwd | faas-cli login --password-stdin
      ```

4. 使用浏览器访问openfaas的ui

   ```
   http://127.0.0.1:31112
   admin/admin123
   ```

> 参考：[https://docs.openfaas.com/deployment/kubernetes/#use-the-ui](https://docs.openfaas.com/deployment/kubernetes/#use-the-ui)

## 2. 使用

> openfaas集成了ci/cd，整个过程就是创建pod，执行命令，返回结果的函数编程。

1. 使用模版创建新函数，语言python3，函数名hello

   ```
   $ faas-cli new --lang python3 hello
   $ tree ./
     ├── build
     │   └── hello
     │       ├── Dockerfile
     │       ├── function
     │       │   ├── handler.py
     │       │   ├── __init__.py
     │       │   └── requirements.txt
     │       ├── index.py
     │       ├── requirements.txt
     │       └── template.yml
     ├── hello
     │   ├── handler.py
     │   ├── __init__.py
     │   └── requirements.txt
     └── hello.yml
   备注: 函数主代码在hello/handler.py中实现。
   $ cat hello.yml
     version: 1.0
     provider:
       name: openfaas
       gateway: http://127.0.0.1:31112
     functions:
       hello:
         lang: python3
         handler: ./hello
         image: hello:latest
     备注： 其中gateway为openfaas路径；image为经过CI后的镜像名
   ```

2. CI 生成handler镜像

   ```
   $ faas-cli build -f hello.yml  # 生成docker镜像
   $ faas-cli push -f hello.yml  # push到指定的docker registry中
   备注: 可选项，--parallel=N， --no-cache， --squash
   ```

3. 部署自己创建的新函数

   ```
   $ export gw=http://$(minikube ip):31112
   $ faas-cli deploy -f hello.yml --gateway $gw
     Deploying: hello.
     No existing function to remove
     Deployed.
     URL: http://192.168.99.100:31112/function/hello
   ```

4. 使用创建的新函数

   ```
   $ echo test | faas-cli invoke hello --gateway $gw
   ```

5. 其它命令

   1. 查看当前部署的函数

      ```
      $ faas-cli list
      ```

   2. 查看openfaas提供的函数

      ```
      $ faas-cli store list
      ```

> 参考: [https://medium.com/faun/getting-started-with-openfaas-on-minikube-634502c7acdf](https://medium.com/faun/getting-started-with-openfaas-on-minikube-634502c7acdf)

## 3. 镜像推送到私有镜像中心

> openfaas在k8s的openfaas-fn的namespace中部署函数，所以，对应的k8s要有该空间的pull权限，需要创建该空间的imagepullsecret，另外，需要在函数的描述manifest中添加imagepullsecret相关字段，比如本例中的hello.yml文件。
>
> 可以将imaeg pull secret绑定到namespace默认的serviceaccount上面，避免每一次修改函数的manifest。

1. 创建imagepullsecret：

   ```bash
   $ kubectl create secret docker-registry my-private-repo \
       --docker-username=$DOCKER_USERNAME \
       --docker-password=$DOCKER_PASSWORD \
       --docker-email=$DOCKER_EMAIL \
       --namespace openfaas-fn
   ```

2. 编辑默认的serviceaccount，添加imagepullsecret字段:

   ```bash
   $ kubectl edit serviceaccount default -n openfaas-fn
   # 底部添加imagepullsecret字段
   # imagePullSecrets:
   # - name: my-private-repo
   ```

> 参考： [openfaas使用私有镜像中心](https://docs.openfaas.com/deployment/kubernetes/#use-the-ui)
