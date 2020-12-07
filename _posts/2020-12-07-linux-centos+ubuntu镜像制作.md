---
layout: post
title: "linux笔记: centos+ubuntu镜像制作"
subtitle: "docker hub上的centos和ubuntu镜像制作"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux

---
## ubuntu镜像

* docker library ubuntu镜像的描述：[https://github.com/docker-library/docs/tree/master/ubuntu](https://github.com/docker-library/docs/tree/master/ubuntu)

* rootfs 的tar包地址： [https://partner-images.canonical.com/core/](https://partner-images.canonical.com/core/)

* dockerfile 地址： [https://github.com/tianon/docker-brew-ubuntu-core/blob/dist-amd64/bionic/Dockerfile](https://github.com/tianon/docker-brew-ubuntu-core/blob/dist-amd64/bionic/Dockerfile)

* utf8:

  ```
  RUN apt-get update && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
  	&& localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
  ENV LANG en_US.utf8
  ```

  



## centos镜像

* 描述：[https://github.com/docker-library/docs/tree/master/centos](https://github.com/docker-library/docs/tree/master/centos)

* rootfs 的tar包制作方式：[https://github.com/CentOS/sig-cloud-instance-build/tree/master/docker](https://github.com/CentOS/sig-cloud-instance-build/tree/master/docker)

  * hard way:

    ```
    # curl http://mirror.centos.org/centos/7/os/x86_64/images/boot.iso -o /tmp/boot7.iso
    # sudo livemedia-creator --make-tar --iso=/tmp/boot7.iso --ks=/path/to/centos-7.ks --image-name=centos-7-docker.tar.xz
    ```

  * easy way:

    ```
    ./containerbuild.sh centos-7.ks
    ```

  有了tar包后，制作镜像：

  * 通过docker import导入：

    ```
    cat centos-version-docker.tar.xz | docker import - container-name
    ```

  * 通过dockerfile构建：

    ```
    FROM scratch
    MAINTAINER you<your@email.here> - ami_creator
    ADD centos-version-docker.tar.xz /
    ```



## 其它

* [https://hub.docker.com/_/ubuntu](https://hub.docker.com/_/ubuntu)
* [https://hub.docker.com/_/centos](https://hub.docker.com/_/centos)



