---
title: Code
type: docs
weight: 3
next: web-server/authentication/
---

Congratulations! Now you could run the project and see the result from your browser. 

There are still somethings to do before we adding any features to the project. Don't worry, we will finish it very quickly.

## Introduce database driver

GoFrame has many [supported databases](https://github.com/gogf/gf/tree/master/contrib/drivers). But to use them, you need to introduce driver into your project first.

Install database driver:

{{< tabs items="MySQL / MariaDB" >}}
{{< tab >}}
``` bash
go get -u github.com/gogf/gf/contrib/drivers/mysql/v2
```
{{< /tab >}}
{{< /tabs >}}

Open `main.go` in the project folder with your favorite editor. Add the following import statement:

{{< tabs items="MySQL / MariaDB" >}}
{{< tab >}}
```go {filename="main.go"}
_ "github.com/gogf/gf/contrib/drivers/mysql/v2"
```

{{% details title="Full `main.go` file" closed="true" %}}
```go {filename="main.go"}
package main

import (
	_ "bootcamp/internal/packed"

	"github.com/gogf/gf/v2/os/gctx"

	"bootcamp/internal/cmd"
    
    _ "github.com/gogf/gf/contrib/drivers/mysql/v2"
)

func main() {
	cmd.Main.Run(gctx.GetInitCtx())
}
```
{{% /details %}}
{{< /tab >}}
{{< /tabs >}}

Then it is finished, you don't need to worry about the driver any more.

## Generate Database Access Object(DAO)

As we mentioned before, `gf` has many powerful code generation functions. Now we will use `gf gen dao`, which was configured before, to generate the DAO related code.

Depend on your previous settings, you may use this command in your docker container or your local machine, and it will generate related code automatically.
```
$ gf gen dao
generated: internal\dao\internal\messages.go
generated: internal\dao\internal\users.go
generated: internal\model\do\messages.go
generated: internal\model\do\users.go
generated: internal\model\entity\messages.go
generated: internal\model\entity\users.go
done!
```

{{< callout type="info" >}}
If there are any changes with database schema, you could run `gf gen dao` again to refresh the DAO related code.
{{< /callout >}}