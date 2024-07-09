---
title: JWT
type: docs
weight: 1
next: web-server/crud/
---

Json Web Token (JWT) 在许多网络应用中都有广泛应用。它是一个无状态，低占用且易于使用的解决方案。

我们将使用一个社区中间件来创建 JWT 服务，查看 [gf-jwt](https://github.com/gogf/gf-jwt) 了解更多内容。你也可以模仿该项目来实现自己的JWT中间件。但为保简洁，让我们先直接使用它。

## 安装

添加 `gf-jwt` 到你的项目：
```bash
go get github.com/gogf/gf-jwt/v2
```

## 创建服务

### 初始化

在你的 `internal/service` 目录下，创建一个 `jwt_auth.go` 文件：

```go {filename="internal/service/jwt_auth.go"}
package service

import (
	jwt "github.com/gogf/gf-jwt/v2"
)

var authService *jwt.GfJWTMiddleware

func JWTAuth() *jwt.GfJWTMiddleware {
	return authService
}
```

### 添加配置

然后我们可以为 jwt 中间件添加初始化配置：

```go {filename="internal/service/jwt_auth.go"}
...
func init() {
	auth := jwt.New(&jwt.GfJWTMiddleware{
		Realm:           "auth zone",
		Key:             []byte("QH7GMFykSVxPrLkc"),
		Timeout:         time.Hour * 1,
		MaxRefresh:      time.Hour * 2,
		IdentityKey:     "uid",
		TokenLookup:     "header: Authorization, query: token",
		TokenHeadName:   "Bearer",
		TimeFunc:        time.Now,
		Authenticator:   Authenticator,
		Unauthorized:    Unauthorized,
		PayloadFunc:     PayloadFunc,
		IdentityHandler: IdentityHandler,
	})
	authService = auth
}

func PayloadFunc(data interface{}) jwt.MapClaims {
	// ToDo
}

func IdentityHandler(ctx context.Context) interface{} {
	// ToDo
}

func Unauthorized(ctx context.Context, code int, message string) {
	// ToDo
}

func Authenticator(ctx context.Context) (interface{}, error) {
	// ToDo
}

```

- **Realm**: 设置展示给用户的 Realm 名称。
- **Key**: 验证 JWT 的密钥。由于会影响所有的 JWT，它不应当被频繁更改。你可以使用 `const` 或者硬编码来存储它。
- **Timeout**: JWT 有效时间。
- **MaxRefresh**: JWT 最大刷新时间。
- **IdentityKey**: JWT 中用来表明用户身份的变量名。还记得我们之前创建的 `users` 表吗？我们应该在这里填入 `uid` 字段。
- **TokenLookup**: 从请求中获取 token 的方法。通过这项设置我们可以从以下位置获取 token：
	- 请求头中的 `Authorization`
	- url 中的请求参数 `token` ，例如 `url?token=xxx`
- **TokenHeadName**: 请求头中 token 前的字符串。例如： `Bearer xxx`.
- **TimeFunc**: 获取当前时间的函数，大部分情况下无需更改。

最后四个函数用于处理认证时的一些过程，我们将会在之后实现他们的具体功能。

### PayloadFunc

PayloadFunc 是在用户登录时调用的回调函数。使用该函数可以为 token 添加自定义的 payload 字段。不过我们在这里直接将原有数据传递到 token 中：
```go {filename="internal/service/jwt_auth.go"}
func PayloadFunc(data interface{}) jwt.MapClaims {
	claims := jwt.MapClaims{}
	params := data.(map[string]interface{})
	if len(params) > 0 {
		for k, v := range params {
			claims[k] = v
		}
	}
	return claims
}
```

### IdentityHandler

IdentityHandler 用于为所有请求获取并设置 JWT 身份信息。在我们的项目中，它的返回值将会设置 `uid` 参数。像这样实现它：
```go {filename="internal/service/jwt_auth.go"}
func IdentityHandler(ctx context.Context) interface{} {
	claims := jwt.ExtractClaims(ctx)
	return claims[authService.IdentityKey]
}
```
现在你就能够在其他地方获取请求的 `uid` 参数了，你也可以在这里添加处理其他数据的逻辑。

### Unauthorized

Unauthorized 用于自定义未授权时的回调函数。通常你可以用它来返回错误消息并进行一些错误处理。
```go {filename="internal/service/jwt_auth.go"}
func Unauthorized(ctx context.Context, code int, message string) {
	r := g.RequestFromCtx(ctx)
	r.Response.WriteJson(g.Map{
		"code":    code,
		"message": message,
	})
	r.ExitAll()
}
```

我们简单地返回了输入的状态码和状态消息，你也可以根据不同的状态码和状态消息来自定义返回内容。

这里使用的包 `g` 是由 `GoFrame` 提供的，它提供了常用类型/函数的定义和耦合调用来创建常用的对象。你可以在[这里](https://pkg.go.dev/github.com/gogf/gf/v2/frame/g)获得更多细节。


{{< callout type="info" >}}
一些 IDE 在保存时会错误的导入 `g` `v1` 的包，而不是 `v2` 的包，你可以直接将它更改为 `github.com/gogf/gf/v2/frame/g`。
{{< /callout >}}

### Authenticator

Authenticator 是 JWT 中间件中最重要的部分，被用于验证登陆参数。它必须返回代表用户身份的用户信息。为保简洁，这里我们将不会检查输入参数，也不会从数据库查找记录。你可以在熟悉删增改查的操作后自行实现完整的逻辑。现在我们像这样设置它：

```go {filename="internal/service/jwt_auth.go"}
func Authenticator(ctx context.Context) (interface{}, error) {
	return g.Map{
		"uid": "0000000000",
		"username": "admin",
	}, nil
}	
```

{{< callout type="info" >}}
确保这里的信息和之前初始化数据库时插入的记录一致，我们将会在删增改查的部分用到他们。
{{< /callout>}}

### 检查完整文件

{{% details title="`jwt_auth.go` 的完整内容" closed="true" %}}
```go {filename="internal/service/jwt_auth.go"}
package service

import (
	"context"
	"time"

	jwt "github.com/gogf/gf-jwt/v2"
	"github.com/gogf/gf/v2/frame/g"
)

var authService *jwt.GfJWTMiddleware

func JWTAuth() *jwt.GfJWTMiddleware {
	return authService
}

func init() {
	auth := jwt.New(&jwt.GfJWTMiddleware{
		Realm:           "auth zone",
		Key:             []byte("QH7GMFykSVxPrLkc"),
		Timeout:         time.Hour * 1,
		MaxRefresh:      time.Hour * 2,
		IdentityKey:     "uid",
		TokenLookup:     "header: Authorization, query: token",
		TokenHeadName:   "Bearer",
		TimeFunc:        time.Now,
		Authenticator:   Authenticator,
		Unauthorized:    Unauthorized,
		PayloadFunc:     PayloadFunc,
		IdentityHandler: IdentityHandler,
	})
	authService = auth
}

func PayloadFunc(data interface{}) jwt.MapClaims {
	claims := jwt.MapClaims{}
	params := data.(map[string]interface{})
	if len(params) > 0 {
		for k, v := range params {
			claims[k] = v
		}
	}
	return claims
}

func IdentityHandler(ctx context.Context) interface{} {
	claims := jwt.ExtractClaims(ctx)
	return claims[authService.IdentityKey]
}

func Unauthorized(ctx context.Context, code int, message string) {
	r := g.RequestFromCtx(ctx)
	r.Response.WriteJson(g.Map{
		"code":    code,
		"message": message,
	})
	r.ExitAll()
}

func Authenticator(ctx context.Context) (interface{}, error) {
	return g.Map{
		"uid":      "0000000000",
		"username": "admin",
	}, nil
}

```
{{% /details %}}

## 测试服务

现在我们已经完成了 JWT 中间件的服务，试一试吧！

在 `internal/controller/hello` 文件夹下有一个默认的控制器，我们可以把它更改为：

```go {filename="internal/controller/hello/hello_v1_hello.go"}
func (c *ControllerV1) Hello(ctx context.Context, req *v1.HelloReq) (res *v1.HelloRes, err error) {
	token, _ := service.JWTAuth().LoginHandler(ctx)
	g.RequestFromCtx(ctx).Response.Writeln(token)
	return
}
```

在你的浏览器中打开 [http://localhost:8000/hello](http://localhost:8000/hello)，你将会看到 JWT 服务生成的 token。

保存一下得到的 token，我们将在下一章中用到它。
