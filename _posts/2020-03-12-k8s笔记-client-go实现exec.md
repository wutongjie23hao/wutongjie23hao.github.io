---
layout: post
title: "client-go实现进入pod使用exec执行命令"
subtitle: "k8s笔记：client-go实现exec"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - k8s
  - kubernetes
---
## 使用client-go执行exec命令

在client-go中没有exec的例子，网上搜索，参考https://github.com/kubernetes/kubernetes/blob/v1.6.1/test/e2e/framework/exec_util.go和https://stackoverflow.com/questions/43314689/example-of-exec-in-k8ss-pod-by-using-go-client 使用golang实现了到pod执行指定命令的代码，如下：

```go
import (
	"bytes"
	"k8s.io/api/core/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/kubernetes/scheme"
	"k8s.io/client-go/rest"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/client-go/tools/clientcmd/api"
	"k8s.io/client-go/tools/remotecommand"
	"strings"
)
func ExecInPod(config *rest.Config, namespace, podName, command, containerName string) (string, string, error) {
	k8sCli, err := kubernetes.NewForConfig(config)
	if err != nil {
		return "", "", err
	}
	cmd := []string{
		"sh",
		"-c",
		command,
	}
	const tty = false
	req := k8sCli.CoreV1().RESTClient().Post().
		Resource("pods").
		Name(podName).
		Namespace(namespace).SubResource("exec").Param("container", containerName)
	req.VersionedParams(
		&v1.PodExecOptions{
			Command: cmd,
			Stdin:   false,
			Stdout:  true,
			Stderr:  true,
			TTY:     tty,
		},
		scheme.ParameterCodec,
	)

	var stdout, stderr bytes.Buffer
	exec, err := remotecommand.NewSPDYExecutor(config, "POST", req.URL())
	if err != nil {
		return "", "", err
	}
	err = exec.Stream(remotecommand.StreamOptions{
		Stdin:  nil,
		Stdout: &stdout,
		Stderr: &stderr,
	})
	if err != nil {
		return "", "", err
	}
	return strings.TrimSpace(stdout.String()), strings.TrimSpace(stderr.String()), err
}
```



这个是非交互式的，交互式的待后面补充。

## QA

1. go mod vendor出现错误，怎么解决？

   修改`go.mod`，如下：

   ```
   k8s.io/api kubernetes-1.15.3
   k8s.io/client-go kubernetes-1.15.3
   ```

   
