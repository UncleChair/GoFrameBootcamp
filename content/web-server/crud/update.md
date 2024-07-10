---
title: Update a message
type: docs
weight: 5
---

Following the previous examples to create your controller, we will focus on `Model` operations from now on.

{{< callout type="info" >}}
In the `RESTful` way, your route should be `PUT /messages/:id`.
{{< /callout >}}

## Update a record

To update the record, we need to know the primary key of it. So the `id` in the route could help us to identify.

### Update

`GoFrame` has a strict rule to use `Update()` function to update a record, you have to add a `where` condition in the method chain, otherwise the operation would not be executed.

{{< tabs items="dao,g.DB" >}}
{{< tab >}}
```go
id := g.RequestFromCtx(ctx).GetRouter("id").Int()
dao.Messages.Ctx(ctx).Where(g.Map{
	dao.Messages.Columns().Id: id,
}).Data(req).Update()
```
{{< /tab >}}
{{< tab >}}
```go
id := g.RequestFromCtx(ctx).GetRouter("id").Int()
g.DB().Model("messages").Where(g.Map{
	dao.Messages.Columns().Id: id,
}).Data(req).Update()
```
{{< /tab >}}
{{< /tabs >}}

### Save

`Save()` is another convenient function to update a record, we have used it in the message creation part. To update a record with it, your data should contain the primary key.


{{< tabs items="dao,g.DB" >}}
{{< tab >}}
```go
id := g.RequestFromCtx(ctx).GetRouter("id").Int()
dao.Messages.Ctx(ctx).Data(g.Map{
	dao.Messages.Columns().Id: id,
	dao.Messages.Columns().UserUid: req.UserUid,
	dao.Messages.Columns().Content: req.Content,
}).Save()
```
{{< /tab >}}
{{< tab >}}
```go
id := g.RequestFromCtx(ctx).GetRouter("id").Int()
g.DB().Model("messages").Data(g.Map{
	dao.Messages.Columns().Id: id,
	dao.Messages.Columns().UserUid: req.UserUid,
	dao.Messages.Columns().Content: req.Content,
}).Save()
```
{{< /tab >}}
{{< /tabs >}}

## Database transaction

You may need to use database transaction to ensure the data consistency, since sometimes you may not want to update records if some operations failed. Here is an example:

```go
// ctx is the context in your controller or other functions.
g.DB().Transaction(ctx, func(ctx context.Context, tx gdb.TX) error {
	// Insert a message
	result, err := tx.Ctx(ctx).Insert("messages", g.Map{
		"user_uid": "0000000000",
		"content": "This is a test message.",
	})
	if err != nil {
		return err
	}
	// Get insert message id
	id, err := result.LastInsertId()
	if err != nil {
		return err
	}
    if id < 10 {
        return gerror.New("New message id must be greater than 10")
    }
	return nil
})
```

In this transaction, first we insert a message in the `messages` table, and then we get the insert id of the message. Finally we check if the id is greater than 10, if not, we return an error.

In the previous three step, every `err` thrown would cause the transaction to rollback. The insert operation in this transaction could only be commit if all the condition is met and not error happened.

{{< callout type="info" >}}
If there is a panic in the transaction, it would also be rolled back to keep the data consistency.
{{< /callout >}}
