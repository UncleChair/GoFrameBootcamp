---
title: Retrieve a message
type: docs
weight: 4
---

After calling the `create message` API, you can simply insert a record in your database. But we still need some functions to return the message in the response. Let's implement it in a `RESTful` way.

## Get a record

### Create API structure

First, let's add a API to get a messages by its `id`:

```go {filename="api/message/v1/show.go"}
package v1

import "github.com/gogf/gf/v2/frame/g"

type Message struct {
	Id      int    `json:"id"`
	UserUid string `json:"user_uid" des:"Message sender ID" eg:"0000000000"`
	Content string `json:"content" des:"Message content" eg:"This is my first message."`
}

type GetMessageReq struct {
	g.Meta `path:"/:id" tags:"Message" method:"get" sum:"Get a message"`
}

type GetMessageRes struct {
	g.Meta  `mime:"application/json"`
	Code    int      `json:"code" v:"required" des:"Status code" eg:"0"`
	Message string   `json:"message" v:"required" des:"Status message" eg:"Success"`
	Data    *Message `json:"data"`
}
```

{{< callout type="info" >}}
Since the `internal` files in Go can not be used out of the folder, we define a `Message` structure here to represent the information of a message. It is good for generating the OpenAPI documentation, but would increase the project complexity. You could also use the `Data` filed with `interface{}` and fill the data in the controller logic with `dao` generated before, but you will lose the detailed return data in your OpenAPI documentation.
{{< /callout >}}

### Add controller logic

Generate controller with `gf gen ctrl` and check it in [Swagger](http://localhost:8000/swagger#tag/Message/paths/~1messages~1:id/get).

Add logic for getting a message in controller:
```go {filename="internal\controller\message\message_v1_get_message.go"}
func (c *ControllerV1) GetMessage(ctx context.Context, req *v1.GetMessageReq) (res *v1.GetMessageRes, err error) {
	id := g.RequestFromCtx(ctx).GetRouter("id").Int()
	g.Dump(id)
	var message *v1.Message
	dao.Messages.Ctx(ctx).Where(g.Map{
		dao.Messages.Columns().Id: id,
	}).Scan(&message)
	res = &v1.GetMessageRes{
		Code:    0,
		Data:    nil,
		Message: "Success",
	}
	if message != nil {
		res.Data = message
	} else {
		res.Code = 1
		res.Message = "No message found"
	}
	return
}
```

Remember how we set the path of the `get` API? It was `/:id`. So now we could get the message id from the router. The `g.Dump(id)` used here is to output stdout of the id, it is a convenient function provided by `GoFrame`.

Then we use the `dao` to get the message by `id`, the scan function here is used to auto-convert the result to the `Message` structure defined in API structure. You could also use other method like `One()` to get the result and fill them in the response.

{{< callout type="info" >}}
Although the `Scan()` is very convenient, you should use it carefully. It is using `reflection` to convert the structure and would consume resources. If you are doing a complex conversion, it is recommended to write an [`UnmarshalValue`](https://goframe.org/pages/viewpage.action?pageId=1114226) to improve the performance.
{{< /callout >}}

The response here would only have a HTTP status code `200`. If you prefer different status code when there is an error, you can add this when doing error handling:

```go
g.RequestFromCtx(ctx).Response.Status = 404
```

This would change the final status code to `404`. Or you could do error handling in your response handler middleware.

{{< callout type="info" >}}
If you are using different HTTP status code, the OpenAPI documentation need to be changed manually. Get the OpenAPI object with `g.Server().GetOpenApi()` and modify it after the server started.
{{< /callout >}}

### Test your API

Since we have registered the route for `message` before, we could just test it:

{{< tabs items="Postman,curl" >}}
{{< tab >}}
GET `http://localhost:8000/messages/1` with your token
{{< /tab >}}
{{< tab >}}
```bash
curl -X GET "http://localhost:8000/messages/1?token=<your token>"
```
{{< /tab >}}
{{< /tabs >}}

And you will get the response:

```json
{
    "code": 0,
    "message": "Success",
    "data": {
        "id": 1,
        "user_uid": "0000000000",
        "content": "This is my first message."
    }
}
```

{{< callout type="info" >}}
If your API returns null, it means that the creation was not succeeded. You may go back to the creation part and check carefully to see where was the error.
{{< /callout >}}

## Get records

It is also very simple to get multiple records. Try create your own API and add this structure to your API definition:

```go
type Messages []*Message
```

And you could also use the `Scan()` to get records:

```go
var messages v1.Messages
dao.Messages.Ctx(ctx).Scan(&messages)
```

Have a test with it, you could get a similar response like this:

```json
{
    "code": 0,
    "message": "Success",
    "data": [
        {
            "id": 1,
            "user_uid": "0000000000",
            "content": "This is my first message."
        }
    ]
}
```

## Get relations (experimental)

Different from other frameworks, `GoFrame` did not provide `hasOne` or other relations. Its principle is to keep the data structure simple and easy to use. But if you want to get the relations, it is also very easy to use the `with` function provided by `GoFrame`.

Go back to the get a `Message` API definition, let's add a `user` structure first:

```go {filename="api/message/v1/show.go"}
type User struct {
	g.Meta   `orm:"table:users"`
	Uid      string `json:"uid" des:"User uid" eg:"0000000000"`
	Username string `json:"username" des:"Username" eg:"admin"`
}
```

We use the `g.Mate` to add the orm tag for the `User` structure, and then we could add relation in message structure:

```go {filename="api/message/v1/show.go"}
type Message struct {
	Id      int    `json:"id"`
	UserUId string `json:"user_uid" des:"Message sender ID" eg:"0000000000"`
	Content string `json:"content" des:"Message content" eg:"This is my first message."`
	User    *User  `orm:"with:uid=user_uid" json:"user" des:"Message sender"`
}
```

The final step to use the relation in `Model` operations is to add a `with` in the method chain:

```go {filename="internal/controller/message/message_v1_get_message.go"}
dao.Messages.Ctx(ctx).WithAll().Where(g.Map{
	dao.Messages.Columns().Id: id,
}).Scan(&message)
```

Now try to get a message from the original api, you would get the addition user information in the response data.

{{< callout type="info" >}}
There are other usage for `with` function that could select certain relations in the methods chain, you could check it [here](https://goframe.org/pages/viewpage.action?pageId=7297190). Also, since it is an experimental function, only retrieve data was implemented now.
{{< /callout >}}
