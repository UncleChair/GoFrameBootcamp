---
title: 添加控制器
type: docs
weight: 1
---

在开始为应用添加逻辑前，我们首先需要创建一个控制器并注册路由。还记得之前章节我们使用过的 `hello` 控制器吗？
{{< filetree/container >}}
  {{< filetree/folder name="api" >}}
    {{< filetree/folder name="hello" >}}
      {{< filetree/folder name="v1" >}}
        {{< filetree/file name="hello.go" >}}
      {{< /filetree/folder >}}
      {{< filetree/file name="hello.go" >}}
    {{< /filetree/folder >}}
  {{< /filetree/folder >}}
  {{< filetree/folder name="internal" >}}
    {{< filetree/folder name="controller" >}}
      {{< filetree/folder name="hello" >}}
        {{< filetree/file name="hello_new.go" >}}
        {{< filetree/file name="hello_v1_hello.go" >}}
        {{< filetree/file name="hello.go" >}}
      {{< /filetree/folder >}}
    {{< /filetree/folder >}}
  {{< /filetree/folder >}}
{{< /filetree/container >}}

路由注册在 `internal/cmd/cmd.go` 文件中：
```go
s.Group("/", func(group *ghttp.RouterGroup) {
    group.Middleware(ghttp.MiddlewareHandlerResponse)
	group.Bind(
	    hello.NewV1(),
	)
})
```

这里使用的方法在 `GoFrame` 中被称为 `规范路由`。这种方法很简单，并且可以结合代码生成命令 `gf gen ctrl` 使用。API 文档也会自动生成，你可以查看 Swagger 页面的 [hello](http://localhost:8000/swagger#tag/Hello/paths/~1hello/get) 来确认。

我们推荐使用 `规范路由`，因为它相较于其他的方式更易于维护和扩展。你也可以在[这里](https://goframe.org/pages/viewpage.action?pageId=1114479)了解到其他的路由注册方式。

## 创建 API 结构体

为了使用代码生成命令 `gf gen ctrl`，你需要先声明 API 结构体。`GoFrame` 要求的文件结构为 `api/{模块}/{版本}/{定义}.go`，并且 API 结构体的名称需要以 `Req` 和 `Res` 结尾。让我们遵循 `RESTful` 的标准来为消息创建一个 API：

```go {filename="api/message/v1/store.go"}
package v1

import "github.com/gogf/gf/v2/frame/g"

type CreateMessageReq struct {
	g.Meta  `path:"/" tags:"Message" method:"post" sum:"创建一条消息"`
	UserUid string `json:"user_uid" v:"required|size:10" des:"消息发送人ID" eg:"0000000000"`
	Content string `json:"content" v:"required|length:1,100" des:"消息内容" eg:"This is my first message."`
}

type CreateMessageRes struct {
	g.Meta  `status:"201" mime:"application/json"`
	Code    int         `json:"code" v:"required" des:"状态码" eg:"0"`
	Message string      `json:"message" v:"required" des:"状态消息" eg:"Success"`
	Data    interface{} `json:"data"`
}

type CreateMessageRes500 struct{
	g.Meta  `resEg:"your_path/example.json"`
}

func (r CreateMessageRes) ResponseStatusMap() map[goai.StatusCode]any {
	return map[goai.StatusCode]any{
		500: CreateMessageRes500{},
	}
}

```
这里使用的 [`g.Meta`](https://pkg.go.dev/github.com/gogf/gf/v2/util/gmeta) 可以为 `请求` 或 `响应` 添加额外的属性。对于这里的请求，`path` 用于定义路由，`tags` 用于设置 `OpenAPI` 文档中的分组，`method` 用于声明接受的 HTTP 方法，而 `sum` 用于添加路由摘要。对于这里的响应，`status` 用于设置默认响应状态码，`mime` 用于设置响应内容的类型。

在这里我们也定义了`500`响应的结构体，其中使用的 `resEg` 标签（`responseExample` 的缩写） 用于设置响应示例文件的路径。目前可以使用的 `example` json文件格式可以为以下两种：

{{< tabs items="Array,Object" >}}
{{< tab >}}
```json {filename="example.json"}
[
  {
    "code": 0,
    "message": "Success",
	"data": null
  },
  {
    "code": 1,
    "message": "Internal Server Error",
	"data": null
  }
]
```
{{< /tab >}}
{{< tab >}}
```json {filename="example.json"}
{
	"success": {
		"code": 0,
		"message": "Success",
		"data": null
	},
	"error": {
		"code": 1,
		"message": "Internal Server Error",
		"data": null
	}
}
```
{{< /tab >}}
{{< /tabs >}}

数组格式将会根据顺序自动添加 `example` 名称，而对象格式则可以手动指定。

只要为响应结构体实现 `goai.IEnhanceResponseStatus` 接口，即添加 `EnhanceResponseStatus` 方法，就可以在文档的生成过程中自动添加额外的响应状态码和响应结构体。如果添加的响应结构体包含任何字段或设置了 `mime` 标签，它将会自动覆盖 `OpenAPI` 文档中的默认内容（一般情况下是自动添加的某些通用响应结构体）。

结构体中的标签不仅可以用于自动生成 `OpenAPI` 文档，还可以用于验证或转换数据。例如，`v` 标签可用于验证请求中的数据。对于我们的请求，`UserUID` 被设置为必需且长度为10，其他字段类似。你可以在[这里](https://goframe.org/pages/viewpage.action?pageId=1114678)找到更多有关数据验证的信息。

这些内容看起来有些不好理解，不过不用担心，在注册路由后你就能清楚地看到它们的功能了。或者你也可以在[这里](https://github.com/gogf/gf/blob/master/util/gtag/gtag.go)找到更多有关 `gtag` 的细节。


{{< callout type="info" >}}
`status` `resEg` `responseExample` 标签和响应结构体的 `ResponseStatusMap` 方法自 `v2.8.0` 版本起可用。
{{< /callout >}}

## 生成控制器

现在我们可以使用 `gf gen ctrl` 基于之前声明的结构体来生成控制器：

```
& gf gen ctrl
generated: ...\api\hello\hello.go
generated: ...\api\message\message.go
generated: internal\controller\message\message.go
generated: internal\controller\message\message_new.go
generated: internal\controller\message\message_v1_create_message.go
done!
```

可以看到它的结构和之前自动生成的 `hello` 控制器类似。

{{< callout type="info" >}}
为了避免每次更改完 `api` 目录下的代码后都要手动运行这个命令，你可以使用一些自动运行插件，例如 VSCode 中的 `Run on Save`。
{{< /callout>}}

打开 `internal\controller\message\message_v1_create_message.go`，你会看到控制器的具体内容：

```go
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	return nil, gerror.NewCode(gcode.CodeNotImplemented)
}
```

这里的响应还没有实现，稍后我们会添加一些代码。

## 注册路由

在生成控制器后，我们需要注册路由。打开 `internal\cmd` 文件夹下的 `cmd.go`，之前我们使用的 `hello` 已经在这里注册了：

```go {filename="internal/cmd/cmd.go"}
s.Group("/", func(group *ghttp.RouterGroup) {
	group.Middleware(ghttp.MiddlewareHandlerResponse)
	group.Bind(
		hello.NewV1(),
	)
})
```

同样，我们可以添加 `message` 的路由：

```go {filename="internal/cmd/cmd.go"}
s.Group("/messages", func(group *ghttp.RouterGroup) {
	group.Middleware(ghttp.MiddlewareHandlerResponse)
	group.Bind(
		message.NewV1(),
	)
})
```

由于之前设置的 `path` 是 `/`，我们需要在注册路由时为路由组添加 `messages`。你也可以选择将 `path` 设置为 `/messages`，这样就无需在路由组中添加了。

{{% details title="`path` 为 `/messages` 时的路由" closed="true" %}}
```go {filename="internal/cmd/cmd.go"}
s.Group("/", func(group *ghttp.RouterGroup) {
	group.Middleware(ghttp.MiddlewareHandlerResponse)
	group.Bind(
		hello.NewV1(),
		message.NewV1(),
	)
})
```
{{% /details %}}

## 测试

好！你已经成功的创建了第一个 API 并注册了它的路由。[这里](http://localhost:8000/swagger#tag/Message)是他的 Swagger 页面。

如果你用过类似 `Postman`的工具，可以使用下面的请求来测试 API：

POST `http://localhost:8000/messages`

```json
{
  "user_uid": "0000000000",
  "content": "This is my first message."
}
```

或者使用 `curl` 来测试 API：

```bash
curl -X POST -H "Content-Type: application/json" -d '{"user_uid":"0000000000","content":"This is my first message."}' "http://localhost:8000/messages"
```

你会看到类似这样的 `Not Implemented` 响应：

```json
{
  "code": 58,
  "message": "Not Implemented",
  "data": null
}
```
