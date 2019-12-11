---
layout: post
title: "docker笔记: docker registry 2.0 Oauth2 Token 授权认证"
subtitle: "docker的Oauth2 Token 认证以及获取token的golang实现"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker
  - linux
---

## 流程
1. 发送请求
2. 在request的header``Www-Authenticate``中提取``realm``、``service``、``scope``，大概样子：
```
Www-Authenticate: Bearer realm="https://auth.docker.io/token",service="registry.docker.io",scope="repository:samalba/my-app:pull,push"
```
3. 使用上面得到的realm、service和scope往授权服务器发送请求，来获取token，大概样子：
```
https://auth.docker.io/token?service=registry.docker.io&scope=repository:samalba/my-app:pull,push
```
默认是post请求，当post不行的时候，使用get请求。返回值大概样子：
```
{"token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiIsImtpZCI6IlBZWU86VEVXVTpWN0pIOjI2SlY6QVFUWjpMSkMzOlNYVko6WEdIQTozNEYyOjJMQVE6WlJNSzpaN1E2In0.eyJpc3MiOiJhdXRoLmRvY2tlci5jb20iLCJzdWIiOiJqbGhhd24iLCJhdWQiOiJyZWdpc3RyeS5kb2NrZXIuY29tIiwiZXhwIjoxNDE1Mzg3MzE1LCJuYmYiOjE0MTUzODcwMTUsImlhdCI6MTQxNTM4NzAxNSwianRpIjoidFlKQ08xYzZjbnl5N2tBbjBjN3JLUGdiVjFIMWJGd3MiLCJhY2Nlc3MiOlt7InR5cGUiOiJyZXBvc2l0b3J5IiwibmFtZSI6InNhbWFsYmEvbXktYXBwIiwiYWN0aW9ucyI6WyJwdXNoIl19XX0.QhflHPfbd6eVF4lM9bwYpFZIV0PfikbyXuLx959ykRTBpe3CYnzs6YBK8FToVb5R47920PVLrh8zuLzdCr9t3w", "expires_in": 3600,"issued_at": "2009-11-10T23:00:00Z"}
```
4. 在请求的头文件中增加授权部分：
```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiIsImtpZCI6IkJWM0Q6MkFWWjpVQjVaOktJQVA6SU5QTDo1RU42Ok40SjQ6Nk1XTzpEUktFOkJWUUs6M0ZKTDpQT1RMIn0.eyJpc3MiOiJhdXRoLmRvY2tlci5jb20iLCJzdWIiOiJCQ0NZOk9VNlo6UUVKNTpXTjJDOjJBVkM6WTdZRDpBM0xZOjQ1VVc6NE9HRDpLQUxMOkNOSjU6NUlVTCIsImF1ZCI6InJlZ2lzdHJ5LmRvY2tlci5jb20iLCJleHAiOjE0MTUzODczMTUsIm5iZiI6MTQxNTM4NzAxNSwiaWF0IjoxNDE1Mzg3MDE1LCJqdGkiOiJ0WUpDTzFjNmNueXk3a0FuMGM3cktQZ2JWMUgxYkZ3cyIsInNjb3BlIjoiamxoYXduOnJlcG9zaXRvcnk6c2FtYWxiYS9teS1hcHA6cHVzaCxwdWxsIGpsaGF3bjpuYW1lc3BhY2U6c2FtYWxiYTpwdWxsIn0.Y3zZSwaZPqy4y9oRBVRImZyv3m_S9XDHF1tWwN7mL52C_IiA73SJkWVNsvNqpJIn5h7A2F8biv_S2ppQ1lgkbw
```

## golang代码实现
```golang
package demo

import (
	"crypto/tls"
	"encoding/json"
	"fmt"
	"net/http"
	"strings"
)
func getToken(realm, service, scope string) string {
	// Bearer realm="https://auth.docker.io/token",service="registry.docker.io",scope="repository:acs/agent:pull"
	postURL := fmt.Sprintf("%s?service=%s&scope=%s", realm, service, scope)
	postReq, err := http.NewRequest("POST", postURL, nil)
	if err != nil {
		panic(err)
	}
	postClient := http.Client{
		Transport: &http.Transport{
			TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
		},
	}
	authResp, err := postClient.Do(postReq)
	if err != nil {
		panic(err)
	}
	if authResp.StatusCode == http.StatusMethodNotAllowed {
		postReq, err := http.NewRequest("GET", postURL, nil)
		if err != nil {
			panic(err)
		}
		authResp, err = postClient.Do(postReq)
		if err != nil {
			panic(err)
		}
	}

	var result map[string]interface{}
	err = json.NewDecoder(authResp.Body).Decode(&result)
	if err != nil {
		panic(err)
	}

	return fmt.Sprintf("%v", result["token"])
}

func parseWWWAutheticateFromResponse(resp *http.Response) (realm, service, scope string) {
	author := resp.Header["Www-Authenticate"]
	bearString := strings.TrimLeft(author[0], "Bearer")
	authString := strings.Split(bearString, ",")
	realm = strings.Trim(strings.Split(authString[0], "=")[1], "\"")
	service = strings.Trim(strings.Split(authString[1], "=")[1], "\"")
	scope = strings.Trim(strings.Split(authString[2], "=")[1], "\"")
	return realm, service, scope
}
```

## reference
[https://docs.docker.com/registry/spec/auth/token/](https://docs.docker.com/registry/spec/auth/token/)
[https://docs.docker.com/registry/spec/auth/oauth/](https://docs.docker.com/registry/spec/auth/oauth/)