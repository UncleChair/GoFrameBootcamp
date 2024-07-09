---
title: Create a message
type: docs
weight: 2
---

After we add the skelton of the controller, we can add some logic to it. First, let's write a simple function to create a message in the database.

To interact with database, we could use `g.DB` provided by `GoFrame` to create model or run some sql. Also those `dao` generated before could also be used to meet our requirement.

{{< callout type="warning" >}}
Models created from `g.DB` and `dao` are different. `g.DB` is not `method chaining` safe, the original model would be affected by those methods. While `dao` is `method chaining` safe, you can use it many times without changing the original model. Be careful of this difference when you are operating models.
{{< /callout>}}

## Saving data

In the controller file we generated before:

{{< tabs items="dao,g.DB" >}}
{{< tab >}}
```go {filename="internal/controller/message/message_v1_create_message.go",hl_lines=[2]}
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	dao.Messages.Ctx(ctx).Save(req)
	return
}
```
{{< /tab >}}
{{< tab >}}
```go {filename="internal/controller/message/message_v1_create_message.go",hl_lines=[2]}
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	g.DB().Model("messages").Save(req)
	return
}
```
{{< /tab >}}
{{< /tabs >}}

`dao` or `g.DB` used here has similar function, just create a model and do some data saving. You may find all methods for `Model` [here](https://pkg.go.dev/github.com/gogf/gf/v2/database/gdb#Model).

{{< callout type="warning" >}}
In most case, you can use `Save()` to insert data into database. But for some database like `clickhouse`, `Save()` method was not supported, you may use `Insert()` instead. Check support methods for databases [here](https://goframe.org/pages/viewpage.action?pageId=1114699)
{{< /callout>}}

## Fill response

Then we could fill some info in our response.

```go {filename="internal/controller/message/message_v1_create_message.go"}
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	dao.Messages.Ctx(ctx).Data(req).Save()
	res = &v1.CreateMessageRes{
		Code:    0,
		Data:    nil,
		Message: "Success",
	}
	return
}
```

Seems the logic is completed right? But actually, some error handling is missing. The `Save` method would return an error if the operation has some problems. So let's add some extra info when this happened.

{{< tabs items="dao,g.DB" >}}
{{< tab >}}
```go {filename="internal/controller/message/message_v1_create_message.go",hl_lines=[2]}
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	_, err = dao.Messages.Ctx(ctx).Data(req).Save()
	res = &v1.CreateMessageRes{
		Code:    0,
		Data:    nil,
		Message: "Success",
	}
	if err != nil {
		res.Code = 1
		res.Message = "Database error happened"
	}
	return
}
```
{{< /tab >}}
{{< tab >}}
```go {filename="internal/controller/message/message_v1_create_message.go",hl_lines=[2]}
func (c *ControllerV1) CreateMessage(ctx context.Context, req *v1.CreateMessageReq) (res *v1.CreateMessageRes, err error) {
	_, err = g.DB().Model("messages").Save(req)
	res = &v1.CreateMessageRes{
		Code:    0,
		Data:    nil,
		Message: "Success",
	}
	if err != nil {
		res.Code = 1
		res.Message = "Database error happened"
	}
	return
}
```
{{< /tab >}}
{{< /tabs >}}

## Check your result

Now you have successfully finished the create message function. Let's have a test.

{{< tabs items="Postman,curl" >}}
{{< tab >}}
POST `http://localhost:8000/messages`

```json
{
  "user_uid": "0000000000",
  "content": "This is my first message."
}
```
{{< /tab >}}
{{< tab >}}
```bash
curl -X POST -H "Content-Type: application/json" -d '{"user_uid":"0000000000","content":"This is my first message."}' "http://localhost:8000/messages"
```
{{< /tab >}}
{{< /tabs >}}

You will see the response from this api, it seems to be ok with calling this API, but the response is very strange:

```json
{
	"code": 0,
	"message": "",
	"data": {
		"code": 0,
		"message": "Success",
		"data": null
	}
}
```

Don't worry, remember how we register our route?

```go {filename="internal/cmd/cmd.go",hl_lines=[2]}
s.Group("your route", func(group *ghttp.RouterGroup) {
	group.Middleware(ghttp.MiddlewareHandlerResponse)
	group.Bind(
		message.NewV1(),
	)
})
```

In this registration, we have added a middleware `ghttp.MiddlewareHandlerResponse` to handle the response. It contains some logic to user our data to generate the final response, which lead to this situation.

Let's create our own middleware to handle the response and add some additional functions to our route in the next part.
