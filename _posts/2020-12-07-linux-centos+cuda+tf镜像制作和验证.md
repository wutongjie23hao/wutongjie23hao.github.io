---
layout: post
title: "linux笔记: centos+cuda+tf镜像制作和验证"
subtitle: "centos7/cuda10.1/cudnn7.6.5/tf2.1镜像制作"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux

---
> 备注：
>
> cuda的docker镜像，启动使用宿主机的驱动，所以，不用安装，只需要安装相应的cuda和cudnn就可以。在容器启动的时候，挂载一下宿主机nvidia驱动库所在目录就行。

## centos7

* [dockerfile地址](https://github.com/CentOS/sig-cloud-instance-images/blob/CentOS-7/docker/Dockerfile)

* 获取rootfs的tar包：

  ```
  wget https://github.com/CentOS/sig-cloud-instance-images/raw/CentOS-7/docker/centos-7-docker.tar.xz
  ```

  

* dockerfile 片段：

  ```
  FROM scratch
  ADD centos-7-docker.tar.xz /
  
  LABEL org.label-schema.schema-version="1.0" \
      org.label-schema.name="CentOS Base Image" \
      org.label-schema.vendor="CentOS" \
      org.label-schema.license="GPLv2" \
      org.label-schema.build-date="20181205"
  
  CMD ["/bin/bash"]
  ```



## cuda10.1

* [dockerfile git地址](https://gitlab.com/nvidia/container-images/cuda/-/tree/master/dist/10.1/centos7-x86_64/base)

* [cuda10.1 基础镜像dockerfile](https://gitlab.com/nvidia/container-images/cuda/-/blob/master/dist/10.1/centos7-x86_64/base/Dockerfile)：

  ```
  FROM centos:7
  RUN NVIDIA_GPGKEY_SUM=d1be581509378368edeec8c1eb2958702feedf3bc3d17011adbf24efacce4ab5 && \
      curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/7fa2af80.pub | sed '/^Version/d' > /etc/pki/rpm-gpg/RPM-GPG-KEY-NVIDIA && \
      echo "$NVIDIA_GPGKEY_SUM  /etc/pki/rpm-gpg/RPM-GPG-KEY-NVIDIA" | sha256sum -c --strict  -
  COPY cuda.repo /etc/yum.repos.d/cuda.repo
  COPY nvidia-ml.repo /etc/yum.repos.d/nvidia-ml.repo
  
  ENV CUDA_VERSION 10.1.243
  ENV CUDA_PKG_VERSION 10-1-$CUDA_VERSION-1
  
  # For libraries in the cuda-compat-* package: https://docs.nvidia.com/cuda/eula/index.html#attachment-a
  RUN yum upgrade -y && yum install -y \
      cuda-cudart-$CUDA_PKG_VERSION \
      cuda-compat-10-1 \
      && ln -s cuda-10.1 /usr/local/cuda \
      && yum clean all \
      && rm -rf /var/cache/yum/*
  
  # nvidia-docker 1.0
  RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf && \
      echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf
  
  ENV PATH /usr/local/nvidia/bin:/usr/local/cuda/bin:${PATH}
  ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64
  
  # nvidia-container-runtime
  ENV NVIDIA_VISIBLE_DEVICES all
  ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
  ENV NVIDIA_REQUIRE_CUDA "cuda>=10.1 brand=tesla,driver>=396,driver<397 brand=tesla,driver>=410,driver<411 brand=tesla,driver>=418,driver<419"
  ```

* [cuda runtime](https://gitlab.com/nvidia/container-images/cuda/-/blob/master/dist/10.1/centos7-x86_64/runtime/Dockerfile):

  ```
  ENV NCCL_VERSION 2.7.8
  
  # setopt flag prevents yum from auto upgrading. See https://gitlab.com/nvidia/container-images/cuda/-/issues/88
  RUN yum install --setopt=obsoletes=0 -y \
      cuda-libraries-$CUDA_PKG_VERSION \
      cuda-nvtx-$CUDA_PKG_VERSION \
      cuda-npp-$CUDA_PKG_VERSION \
      libcublas10-10.2.1.243-1 \
      libnccl-2.7.8-1+cuda10.1 \
      && yum clean all \
      && rm -rf /var/cache/yum/*
  
  RUN yum install -y yum-plugin-versionlock && yum versionlock libcublas10
  ```

* [cudnn7 library](https://gitlab.com/nvidia/container-images/cuda/-/blob/master/dist/10.1/centos7-x86_64/runtime/cudnn7/Dockerfile):

  ```
  ENV CUDNN_VERSION 7.6.5.32
  LABEL com.nvidia.cudnn.version="${CUDNN_VERSION}"
  
  RUN yum install -y \
      libcudnn7-${CUDNN_VERSION}-1.cuda10.1 \
      && yum clean all \
      && rm -rf /var/cache/yum/*
  ```

  

## 使用tf进行验证

1. 安装tf

   ```
   pip install --upgrade pip
   pip install --upgrade setuptools
   ## 最新tf2.1.0
   pip install tensorflow-gpu
   ```

2. 验证

   ```
   python -c 'import tensorflow.compat.v1 as tf; import os; tf.disable_v2_behavior();os.environ["CUDA_VISIBLE_DEVICES"] ="0"; sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))'
   
   python -c "import tensorflow as tf; tf.test.is_gpu_available(cuda_only=False,min_cuda_compute_capability=None)"
   ```



## 容器启动命令

```
docker run \
--cap-add=NET_ADMIN \
--cap-add=SYS_ADMIN \
-it \
-v <nvidia_driver>:/usr/local/nvidia:ro \
tf2.1 
```


