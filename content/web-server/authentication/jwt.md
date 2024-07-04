---
title: JWT
type: docs
weight: 1
next: web-server/crud/
---

Json Web Token (JWT) is wildly used in many web applications. It is status free, low cost and easy to use.

We will use a community middleware to create our JWT service. See [gf-jwt](https://github.com/gogf/gf-jwt) for details. You could also mimic that project to implement your own JWT middleware. But to be simple, let's just use it first.

## Install

Add `gf-jwt` to our project:
```bash
go get github.com/gogf/gf-jwt/v2
```

## Create service

### Initial

In your `internal/service` directory, create a `jwt_auth.go` file:

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

### Add configurations

Then we could add configurations to jwt middleware:

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

- **Realm**: Set Realm name to display to the user, change it as your wish.
- **Key**: Secret key used to validate JWT, it should not be changed frequently since it will affect all JWT. You could use `const` to store it or just hard code it.
- **Timeout**: The duration of the JWT.
- **MaxRefresh**: The maximum refresh time of a JWT.
- **IdentityKey**: The key name of the identity in the JWT. Remember our `users` table created before? We should fill it with `uid`.
- **TokenLookup**: The way to get the token from the request. With this settings we could get token from:
	- `Authorization` in header
	- `token` in query, like `url?token=xxx`
- **TokenHeadName**: String before token in header. For example: `Bearer xxx`.
- **TimeFunc**: The function to get current time. In most cases you don't need to change it.

And the final four functions are used to handle some processes. We would implement those todo content in the next few steps.

### PayloadFunc

PayloadFunc is a callback function that will be called during login. Using this function is possible to add additional payload data to the token. We could just use all the data it passed without adding parameters like this:
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

IdentityHandler get the identity from JWT and set the identity for every request. Its return would set the parameter, which is `uid` in our project. Implement it like this:
```go {filename="internal/service/jwt_auth.go"}
func IdentityHandler(ctx context.Context) interface{} {
	claims := jwt.ExtractClaims(ctx)
	return claims[authService.IdentityKey]
}
```
Now you are able to get parameter `uid` from every request in other place. You can also add other handle logic in this function to deal with other data.

### Unauthorized

Unauthorized is used to define customized unauthorized callback function. Usually, you could use it to return customized error message and do some error handling.
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
We simply return the code and message from the input, you could customize return data based on code number or message content.

The package `g` we using here is provided by `GoFrame` to offer commonly used type/function defines and coupled calling for creating commonly used objects. You could find more details in [g](https://pkg.go.dev/github.com/gogf/gf/v2/frame/g).


{{< callout type="info" >}}
Sometimes IDE would not auto import `g` `v2` but `v1` package on saving, you can just change it to `github.com/gogf/gf/v2/frame/g`
{{< /callout >}}

### Authenticator

Authenticator is used to validate login parameters, which is the most important part of the middleware. It must return user data as user identifier. To keep it simple, we would not check input and get data from database. You may implement full logic after you get familiar with the CRUD operations in the next chapter. For now, you can just set it like this:
```go {filename="internal/service/jwt_auth.go"}
func Authenticator(ctx context.Context) (interface{}, error) {
	return g.Map{
		"uid": "0000000000",
		"username": "admin",
	}, nil
}	
```

{{< callout type="info" >}}
Keep these info the same as those created in database initialization part, we would use them in CRUD operations.
{{< /callout>}}

### Check full file

{{% details title="Full file of `jwt_auth.go`" closed="true" %}}
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

## Test service

Now we have finished JWT middleware service, have a test with it!

There is a controller created by default in `internal/controller/hello` folder, we could just change it to:

```go {filename="internal/controller/hello/hello_v1_hello.go"}
func (c *ControllerV1) Hello(ctx context.Context, req *v1.HelloReq) (res *v1.HelloRes, err error) {
	token, _ := service.JWTAuth().LoginHandler(ctx)
	g.RequestFromCtx(ctx).Response.Writeln(token)
	return
}
```

Open your browser with [http://localhost/hello](http://localhost/hello), you will see the token generated from JWT service.

Save the token for a while, we may use it in the next chapter.
