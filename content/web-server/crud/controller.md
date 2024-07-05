---
title: Add a controller
type: docs
weight: 1
---

Before we add any logic to our application, first we need a controller and its route. Remember the `hello` controller? You have used it in the previous chapter.
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

This method using here is call `Standardized Routing` in `GoFrame`, which is easy to use and could be combined with code generation command `gf gen ctrl`. The API documentation would also be generated automatically, see Swagger page [hello](http://localhost:8000/swagger#tag/Hello/paths/~1hello/get).

We recommend to use `Standardized Routing` since it would be better to maintain and extend. You could also find other ways of routing [here](https://goframe.org/pages/viewpage.action?pageId=1114479).

## Create API structure

## Generate Controller

## Register route

## Test your result
