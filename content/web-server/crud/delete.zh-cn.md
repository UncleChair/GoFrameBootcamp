---
title: 删除消息
type: docs
weight: 6
next: web-server/deploying/
---

类似 `Update()`，在使用 `Delete()` 时也需要在方法链中带上 `where` 条件。

`GoFrame` 提供了非常方便的自动填充插入、更新和删除时间字段的功能。你需要遵循以下步骤来使用这一默认功能：
- 时间字段需要允许 `NULL` 值。
- 时间字段的类型必须是时间相关的，例如 `datetime`，`timestamp` 或 `date`。
- 时间字段应当被命名为 `created_at`，`updated_at` 或 `deleted_at`。

你也可以在 `config.yaml` 的数据库配置中添加如下配置:

```yaml {filename="manifest/config/config.yaml"}
database:
  default:
    createdAt: （可选）更改创建时间字段名称。
    updatedAt: （可选）更改更新时间字段名称。
    deletedAt: （可选）更改删除时间字段名称。
    timeMaintainDisabled: （可选）是否关闭时间维护特性，当设置为 true 时，上面所有时间字段的设置都会无效。
```

因为我们已经在数据库初始化时添加了相关字段，所以这里无需其他更改就可以使用。

## 软删除

软删除是一种允许你删除记录但不完全从数据库删除记录的方法。实际上它会在数据库中设置一个标记来表示某条记录已经被删除了。这样就可以保留删除记录的历史记录，同时保证数据的完整性。

{{< callout type="warning" >}}
软删除仅在使用链式调用并且相应的表中有 `deletedAt` 相关字段时才会生效。
{{< /callout >}}

删除第一条消息：

{{< tabs items="dao,g.DB" >}}
{{< tab >}}
```go
dao.Messages.Ctx(ctx).Delete("id", 1)
```
{{< /tab >}}
{{< tab >}}
```go
g.Model("messages").Delete("id", 1)
```
{{< /tab >}}
{{< /tabs >}}

然后尝试再次获取它：

{{< tabs items="dao,g.DB" >}}
{{< tab >}}
```go
dao.Messages.Ctx(ctx).Where("id", 1).One()
```
{{< /tab >}}
{{< tab >}}
```go
g.Model("messages").Where("id", 1).One()
```
{{< /tab >}}
{{< /tabs >}}

你会发现现在已经找不到记录了。

要在软删除后获取记录，我们可以在方法链中添加 `Unscoped()`：

{{< tabs items="dao,g.DB" >}}
{{< tab >}}
```go
dao.Messages.Ctx(ctx).Unscoped().Where("id", 1).One()
```
{{< /tab >}}
{{< tab >}}
```go
g.Model("messages").Unscoped().Where("id", 1).One()
```
{{< /tab >}}
{{< /tabs >}}

然后你就可以看到结果了。你也可以在数据库中查看，第一条记录的 `deleted_at` 字段现在已经不再是空值了。

{{< callout type="info" >}}
可以使用 `g.Dump` 在测试时输出标准输出。
{{< /callout >}}

## 硬删除

硬删除是从数据库中永久删除记录的方法。

如果你没有设置 `deletedAt` 相关字段，那么所有的删除操作都会是硬删除。如果你已经使用了软删除并且想要永久删除记录，你也可以在方法链中添加 `Unscoped()`。

{{< tabs items="dao,g.DB" >}}
{{< tab >}}
```go
dao.Messages.Ctx(ctx).Unscoped().Delete("id", 1)
```
{{< /tab >}}
{{< tab >}}
```go
g.Model("messages").Unscoped().Delete("id", 1)
```
{{< /tab >}}
{{< /tabs >}}

这样你就再也不能找到这条数据了。
