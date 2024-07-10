---
title: 更新消息
type: docs
weight: 5
---

根据前几个案例相同的步骤创建你的控制器，接下来我们将专注于 `Model` 相关的操作。

{{< callout type="info" >}}
按照 `RESTful` 的风格，你的路由应当是 `PUT /messages/:id`。
{{< /callout >}}

## 更新一条记录

要更新记录，我们需要知道这条记录的主键。在路由中的 `id` 参数可以帮我们做到这点。

### Update

`GoFrame` 对于使用 `Update()` 更新记录有着严格的限制，你必须在方法链中添加一个 `where` 条件，否则操作将不会被提交执行。

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

我们之前在创建消息部分使用的 `Save()` 是另一个能够方便地更新记录的方法。如果要用它来更新记录，需要在更新的数据中带上主键。

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

## 数据库事务

你可能需要用到数据库事务来保证数据的一致性，因为有时在某些操作失败后，你可能会不想继续更新记录。下面是一个示例：

```go
// ctx 是你在控制器或者其他函数中传递的 context。
g.DB().Transaction(ctx, func(ctx context.Context, tx gdb.TX) error {
	// 插入一条数据
	result, err := tx.Ctx(ctx).Insert("messages", g.Map{
		"user_uid": "0000000000",
		"content": "This is a test message.",
	})
	if err != nil {
		return err
	}
	// 获取插入数据的id
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

在这个事务中，首先我们在 `messages` 表中插入一条数据，然后获取插入数据的 id。最后我们检查 id 是否大于10，如果不大于10，我们就返回一个错误。

在以上的三个步骤中，任何抛出的 `err` 都会触发最终的事务回滚。事务中的插入数据操作只有在所有条件都满足并且没有发生错误时才能被提交执行。

{{< callout type="info" >}}
如果事务中存在 panic，为了确保数据的一致性，事务也会回滚。
{{< /callout >}}
