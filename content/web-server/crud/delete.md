---
title: Delete a message
type: docs
weight: 6
next: web-server/deploying/
---

Similar to `Update()`, you also have to add a `where` condition in the method chain to use `Delete()`.

`GoFrame` offered a convenient feature to auto fill the insert, update and delete time field. To use it by default, you need to follow these rules:
- Time filed should allow `NULL` value.
- Time field type should be time related, like `datetime`, `timestamp` or `date`.
- Time field should be named as `created_at`, `updated_at` or `deleted_at`.

You could also set this configuration in `config.yaml` database:

```yaml {filename="manifest/config/config.yaml"}
database:
  default:
    createdAt: (optional) Change your create time field here.
    updatedAt: (optional) Change your update time field here.
    deletedAt: (optional) Change your delete time field here.
    timeMaintainDisabled: (optional) Disable the time maintain function, when set to true, all time filed would not work.
```

Since we have add related filed in database initialization part, we don't need to change anything with using it.

## Soft delete

Soft delete is a feature that allows you to delete a record without actually deleting it from the database. Instead, it sets a flag in the database indicating that the record has been deleted. This allows you to keep a history of deleted records while still maintaining the integrity of your data.

{{< callout type="warning" >}}
Soft delete is only functional when using method chain and the related table has `deletedAt` field.
{{< /callout >}}

Delete the first message with:

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

Then try to get it again with:

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

You will find there is no record now.

To get the record after soft delete, we could add `Unscoped()` in the method chain:

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

Then you can see the result. You could also check in the database, the `deleted_at` field for the first message is not null now.

{{< callout type="info" >}}
You could use `g.Dump` to output stdout when testing.
{{< /callout >}}

## Hard delete

Hard delete is a permanent deletion, it will delete the record from the database.

If you have not set the `deletedAt` filed, all the deletion would be hard delete by default. If you have used the soft delete and want to delete record permanently, you could also use `Unscoped()` in the method chain.

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

Then you could not find that record anymore.
