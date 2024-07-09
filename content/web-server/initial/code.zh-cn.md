---
title: 代码
type: docs
weight: 3
next: web-server/authentication/
---

恭喜！现在你已经可以运行项目并在浏览器中看到结果了。

不过在开始为项目添加其他功能之前，我们还需要做一些别的工作。别担心，很快就能完成。

## 导入数据库驱动

GoFrame 支持[多种类型的数据库](https://github.com/gogf/gf/tree/master/contrib/drivers)。但你需要在使用它们前为项目导入对应的驱动。

安装数据库驱动：

{{< tabs items="MySQL / MariaDB" >}}
{{< tab >}}
``` bash
go get -u github.com/gogf/gf/contrib/drivers/mysql/v2
```
{{< /tab >}}
{{< /tabs >}}

使用你喜欢的编辑器打开项目中的 `main.go` 文件，添加以下内容：

{{< tabs items="MySQL / MariaDB" >}}
{{< tab >}}
```go {filename="main.go"}
_ "github.com/gogf/gf/contrib/drivers/mysql/v2"
```

{{% details title="完整的 `main.go` 文件" closed="true" %}}
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

不用担心后续的数据库使用，这样驱动的导入就完成了。

## 生成数据访问对象（DAO）

我们曾提过 `gf` 有很多强大的代码生成功能。现在我们将使用之前配置好的 `gf gen dao` 来生成数据访问对象。

根据之前设置的不同，你可以在docker容器内或本地机器中运行该命令，它将自动生成相关代码。
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
如果数据库结构有任何变化，你可以再次运行 `gf gen dao` 来刷新 DAO 相关的代码。
{{< /callout >}}