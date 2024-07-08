---
title: Add a controller
type: docs
weight: 1
---

Before we adding any logic to our application, first we need a controller and its route. Remember the `hello` controller? You have used it in the previous chapter.
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

The route is registered in `internal/cmd/cmd.go`:
```go
s.Group("/", func(group *ghttp.RouterGroup) {
    group.Middleware(ghttp.MiddlewareHandlerResponse)
	group.Bind(
	    hello.NewV1(),
	)
})
```

This method using here is called `Standardized Routing` in `GoFrame`, which is easy to use and could be combined with code generation command `gf gen ctrl`. The API documentation would also be generated automatically, see Swagger page [hello](http://localhost:8000/swagger#tag/Hello/paths/~1hello/get).

We recommend to use `Standardized Routing` since it would be better to maintain and extend. You could also find other ways of routing [here](https://goframe.org/pages/viewpage.action?pageId=1114479).

## Create API structure

To use the code generation command `gf gen ctrl`, you need to declare the API structure first. `GoFrame` require certain file structure to generate controller, which is `api/{model}/{version}/{definition}.go`. And the API structure name is also require to end with `Req` and `Res`. Let's create a new API in `RESTful` way for our message:

```go {filename="api/message/v1/store.go"}
package v1

import "github.com/gogf/gf/v2/frame/g"

type CreateMessageReq struct {
	g.Meta  `path:"/" tags:"Message" method:"post" sum:"Create a message"`
	UserUId string `json:"user_uid" v:"required|size:10" des:"Message sender ID" eg:"0000000000"`
	Content string `json:"content" v:"required|length:1,100" des:"Message content" eg:"This is my first message."`
}

type CreateMessageRes struct {
	g.Meta  `mime:"application/json"`
	Code    int         `json:"code" v:"required" des:"Status code" eg:"0"`
	Message string      `json:"message" v:"required" des:"Status message" eg:"Success"`
	Data    interface{} `json:"data"`
}
```

The [`g.Meta`](https://pkg.go.dev/github.com/gogf/gf/v2/util/gmeta) here is used to add additional attributes to `Request` or `Response`. For this request, `path` is used to define the route, `tags` is used to set group in `OpenAPI` documentation, `method` is used to declare the accept HTTP method and `sum` is used to add summary to the route. And for the response, `mime` is just used to set the response content type.

Tags in structure could not only be used to generate `OpenAPI` documentation, but also to validate data or transform data. For example, `v` tag could be used to validate data in request. For our request here, the `UserUID` is set to be required and the length should be 10, similar for other fields. Documentation for validation [here](https://goframe.org/pages/viewpage.action?pageId=1114678).

Those tags seems a little confusing, but don't worry, you will see their function clearly soon after we register the route. Or you could also find more details for `gtag` [here](https://github.com/gogf/gf/blob/master/util/gtag/gtag.go).

## Generate Controller

Now we could use `gf gen ctrl` to generate controller with those structures declared before:

```
& gf gen ctrl
generated: ...\api\hello\hello.go
generated: ...\api\message\message.go
generated: internal\controller\message\message.go
generated: internal\controller\message\message_new.go
generated: internal\controller\message\message_v1_create_message.go
done!
```

And you will see its structure is similar to the `hello` controller auto-generated before.

{{< callout type="info" >}}
To avoid running this command every time after changing some code in `api` directory, you could use some auto-run plugin, like `Run on Save` in VSCode.
{{< /callout>}}

Open the `internal\controller\message\message_v1_create_message.go`, you will see the implementation of controller here:

```go
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	return nil, gerror.NewCode(gcode.CodeNotImplemented)
}
```

The response is not implemented yet, we may add some code later.

## Register route

After generate controller, we need to register the route. Open the `cmd.go` in `internal\cmd` folder, the `hello` we used before has been registered here:

```go {filename="internal/cmd/cmd.go"}
s.Group("/", func(group *ghttp.RouterGroup) {
	group.Middleware(ghttp.MiddlewareHandlerResponse)
	group.Bind(
		hello.NewV1(),
	)
})
```

We could add the `message` similarly:

```go {filename="internal/cmd/cmd.go"}
s.Group("/message", func(group *ghttp.RouterGroup) {
	group.Middleware(ghttp.MiddlewareHandlerResponse)
	group.Bind(
		message.NewV1(),
	)
})
```

Since the `path` we set before is `/`, we need to add `message` to the route group when registering. You could also set the path in `api` structure to `/message`, and you can ignore the `message` here.

{{% details title="route used for controller if api `path` is set to `/message`" closed="true" %}}
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

## Test your result

Good job! You have successfully created your first API and registered the route. Check it in Swagger [here](http://localhost:8000/swagger#tag/Message).

If you have used tools like `Postman`, you could test the API with the following request:

POST `http://localhost:8000/message`

```json
{
  "user_uid": "0000000000",
  "content": "This is my first message."
}
```

Or use `curl` to test the API:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"user_uid":"0000000000","content":"This is my first message."}' "http://localhost:8000/message"
```

You will see the `Not Implemented` response like this:

```json
{
  "code": 58,
  "message": "Not Implemented",
  "data": null
}
```
