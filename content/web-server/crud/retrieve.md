---
title: Retrieve a message
type: docs
weight: 4
---

After calling the `create message` API, you can simply insert a record in your database. But we still need some functions to return the message in the response. Let's implement it in a `RESTful` way.

## Get a record

First, let's add a API to get a messages by its `id`:

```go {filename="api/message/v1/show.go"}
package v1

import "github.com/gogf/gf/v2/frame/g"

type Message struct {
	Id      int    `json:"id"`
	UserUId string `json:"user_uid" des:"Message sender ID" eg:"0000000000"`
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
Since the `internal` files in Go can not be used out of the folder, we define a `Message` structure here to represent the information of a message. It is good for generating the OpenAPI documentation, but would increase the project complexity, since every API would need a full detailed structure of its response. You could also use the `Data` filed with `interface{}` and fill the data in the controller logic with `dao` generated before, but you will lose the detailed return data in your OpenAPI documentation.
{{< /callout >}}

Generate controller with `gf gen ctrl` and check it in [Swagger](http://localhost:8000/swagger#tag/Message/paths/~1messages~1:id/get).

Add logic for getting a message in controller:
```go {filename="internal\controller\message\message_v1_get_message.go"}
func (c *ControllerV1) GetMessage(ctx context.Context, req *v1.GetMessageReq) (res *v1.GetMessageRes, err error) {
	id := g.RequestFromCtx(ctx).GetRouter("id").Int()
	g.Dump(id)
	var message *v1.Message
	dao.Users.Ctx(ctx).Where(g.Map{
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

Remember how we set the path of the `get` API? It was `/:id`. So now we could get the message id from the router.

Since we have registered the route for `message` before, we could just test this API:

{{< tabs items="Postman,curl" >}}
{{< tab >}}
GET `http://localhost:8000/messages/1` with your token

```json
{
  "user_uid": "0000000000",
  "content": "This is my first message."
}
```
{{< /tab >}}
{{< tab >}}
```bash
curl -X GET -H "Content-Type: application/json" "http://localhost:8000/messages/1?token=<your token>"
```
{{< /tab >}}
{{< /tabs >}}

## Get records

## Use a template

## Get relations (experimental)
