---
title: Cookie 与 Session
type: docs
weight: 2
prev: false
next: web-server/crud/
---

`Cookie` 和 `Session` 是用于用户验证的传统方案，通常我们会将它们结合使用来保证更高的安全性。`GoFrame` 为此提供了非常方便的管理方法。`SessionId` 默认通过 `Cookie` 传递，也可以在设置后通过请求头传递。

## Cookie

`GoFrame` 的 `Cookie` 变量默认有效时间为1年，你也可以在设置变量时自行更改有效时长。


在 `internal/controller/hello` 文件夹下有一个默认创建的控制器（controller），我们可以用它来测试 cookie 的用法：

```go {filename="internal/controller/hello/hello_v1_hello.go"}
func (c *ControllerV1) Hello(ctx context.Context, req *v1.HelloReq) (res *v1.HelloRes, err error) {
	r := g.RequestFromCtx(ctx)
	sessionID := "SessionID"
	r.Cookie.SetSessionId(sessionID)
	r.Response.Writeln("Set session id success")
	return
}
```

我们这里通过上下文获取到了`请求`对象并且在cookie中设置了session id。

打开浏览器，开启开发者工具并打开链接：[http://localhost:8000/hello](http://localhost:8000/hello).

你将会在请求头中看到 `Set-Cookie`：

```
Set-Cookie: gfsessionid=SessionID; Path=/; Expires=<your expire time>
```

本质上 `Cookie` 的 `SetSessionId` 方法只是将session id赋值给了 `gfsessionid`，你也可以使用 `Set()` 方法来设置其他的键值对。[这里](https://pkg.go.dev/github.com/gogf/gf/v2/net/ghttp#Cookie)有一些其他的 `Cookie` 方法。

{{< callout >}}
要让 cookie 随着用户的浏览会话过期，我们需要使用 `SetCookie` 设置键值对并将 `maxAge` 设为 `0`。
```go
r.Cookie.SetCookie("MyCookieKey", "MyCookieValue", "", "/", 0)
```
{{< /callout >}}

## Session

`GoFrame` 提供了用于管理 session 的包 `gsession`，根据存储 `Session` 方式的不同分为四种类型：

| 存储方式 | 支持分布式 | 支持持久化 | 内存占用 | 执行效率 |
| --- | --- | --- | --- | --- |
| `StorageFile` | 否 | 是 | 中 | 中 |
| `StorageMemory` | 否 | 否 | 高 | 高 |
| `StorageRedis` | 是 | 是 | 中 | 中 |
| `StorageRedisHashTable` | 是 | 是 | 低 | 低 |

文件存储模式适用于大多数情况，因此我们首先从它开始。你也可以在[这里](https://pkg.go.dev/github.com/gogf/gf/v2/os/gsession)找到有关 `gsession` 包的详细信息。

`Session` 的默认有效时长为 `24` 小时，你也可以在初始化服务的时候设置它：
```go
s := g.Server()
s.SetSessionMaxAge(time.Hour)
```

{{< callout type="warning" >}}
由于文件模式是存储 session 的默认模式，我们并不需要手动设置存储方法。不过在其他情况下，我们需要通过 `s.SetSessionStorage()` 设置存储方式。
{{< /callout >}}

{{% steps %}}

### 生成 session id

在前面的部分，我们使用了硬编码的字符串用以测试 session id，现在我们需要实现生成唯一 session id 的方法。我们可以使用 `GoFrame` 提供的 `guid` 方法来生成 uid，添加一个函数：

```go
func GenerateSessionId(r *ghttp.Request) string {
	var (
		address = r.RemoteAddr
		header  = fmt.Sprintf("%v", r.Header)
	)
	return guid.S([]byte(address), []byte(header))
}
```

### 设置 session id

然后我们可以更改之前用过的 `hello` 控制器的方法：

```go
func (c *ControllerV1) Hello(ctx context.Context, req *v1.HelloReq) (res *v1.HelloRes, err error) {
	r := g.RequestFromCtx(ctx)
	sessionID := GenerateSessionId(r)
	r.Cookie.SetSessionId(sessionID)
	r.Session.SetId(SessionId)
	r.Session.Set("SessionId "+sessionID, "Here is your secret data")
	r.Response.Write(r.Session.Data())
	return
}
```

{{< callout type="info" >}}
需要注意的是，使用 `r.Cookie.SetSessionId()` 方法并不会改变当前的 session id，尝试用 `r.Session.Id()` 方法获取id，你会发现它和我们当前生成的并不一致。
{{< /callout >}}

再次测试这个链接 [http://localhost:8000/hello](http://localhost:8000/hello)，你会发现session id会产生变化并且能够获取到对应 session 的数据。

{{% /steps %}}

{{< callout type="info" >}}
使用 `r.Session.RemoveAll()` 来删除所有与请求的 session 有关的数据。
{{< /callout >}}

你可以用上面提到的方法，通过 `Cookie` 和 `Session` 来设计并创建自己的认证系统。
