---
title: Cookie & Session
type: docs
weight: 2
prev: false
next: web-server/crud/
---

`Cookie` and `Session` are traditional solutions used to authenticate users. Usually, they are combined to provide a better security. `GoFrame` provides useful functions to manage them.

In any condition, you could use `ghttp.Request` to get the `Cookie` or `Session` object. The `Cookie` object don't require any manual `Close`, it will be auto closed when the request is finished. The `SessionId` is passed through the `Cookie` by default, and could be set to pass through header.

## Cookie

The default timeout for `Cookie` variable is 1 year in `GoFrame`, you could also give a custom timeout when you are setting a variable.

There is a controller created by default in `internal/controller/hello` folder, we could try the cookie with it:

```go {filename="internal/controller/hello/hello_v1_hello.go"}
func (c *ControllerV1) Hello(ctx context.Context, req *v1.HelloReq) (res *v1.HelloRes, err error) {
	r := g.RequestFromCtx(ctx)
	sessionID := "SessionID"
	r.Cookie.SetSessionId(sessionID)
	r.Response.Writeln("Set session id success")
	return
}
```

In this function, we get the `Request` object from context and set the session ID in the cookie.

Open the browser, turn on the developer tool and visit this link: [http://localhost:8000/hello](http://localhost:8000/hello).

You will find the `Set-Cookie` in header:
```
Set-Cookie: gfsessionid=SessionID; Path=/; Expires=<your expire time>
```

Basically the `SetSessionId` function for `Cookie` is just set the session id to key `gfsession`, you could also use `Set()` functions to set other key-value pairs. You could also find other functions for `Cookie` [here](https://pkg.go.dev/github.com/gogf/gf/v2/net/ghttp#Cookie).

{{< callout >}}
To set the cookie to expire with the user's browsing session, we could use `SetCookie` function to set key-value pair and set the `maxAge` to `0`.
```go
r.Cookie.SetCookie("MyCookieKey", "MyCookieValue", "", "/", 0)
```
{{< /callout >}}

## Session

`GoFrame` provides a useful session management package `gsession`. It can be divided into the following four types according to the different storage method with `Session`:

| Storage | Distributed system | Permanent | Memory usage | Efficiency |
| --- | --- | --- | --- | --- |
| `StorageFile` | No | Yes | Medium | Medium |
| `StorageMemory` | No | No | High | High |
| `StorageRedis` | Yes | Yes | Medium | Medium |
| `StorageRedisHashTable` | Yes | Yes | Low | Low |

The file storage mode is suitable for many cases, so we may start with that. You could also find more details for `gsession` package [here](https://pkg.go.dev/github.com/gogf/gf/v2/os/gsession).

The default timeout for `Session` is `24` hour, you could also set it with server object when initializing:
```go
s := g.Server()
s.SetSessionMaxAge(time.Hour)
```

{{< callout type="warning" >}}
Since file mode is the default mode of session storage, we don't need to set storage method manually. In other cases, you may need to set storage with `s.SetSessionStorage()`.
{{< /callout >}}


{{% steps %}}

### Generate session id

In the previous part, we are using hard coded string for session id, which is just for demo. Now we should implement a function to generate a unique session id. `GoFrame` provides a function `guid` that could be used to generate uid, add a function with it:

```go
func GenerateSessionId(r *ghttp.Request) string {
	var (
		address = r.RemoteAddr
		header  = fmt.Sprintf("%v", r.Header)
	)
	return guid.S([]byte(address), []byte(header))
}
```

### Set session id
Then we could modify the function in the `hello` controller used before:

```go
func (c *ControllerV1) Hello(ctx context.Context, req *v1.HelloReq) (res *v1.HelloRes, err error) {
	r := g.RequestFromCtx(ctx)
	sessionID := GenerateSessionId(r)
	r.Cookie.SetSessionId(sessionID)
	r.Session.Set("SessionId "+sessionID, "Here is your secret data")
	r.Response.Write(r.Session.Data())
	return
}
```

Now test this link [http://localhost:8000/hello](http://localhost:8000/hello) again, you will find the session id would change and you could get data from related session.

{{% /steps %}}

{{< callout type="info" >}}
Use `r.Session.RemoveAll()` to remove all related data from the request session.
{{< /callout >}}

You could use these functions mentioned above to design and create your own authentication system with `Cookie` and `Session`.
