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

In the controller file we generated before:
```go

```