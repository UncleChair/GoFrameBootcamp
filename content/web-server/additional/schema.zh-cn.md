---
title: 数据库结构管理
type: docs
weight: 2
---

大部分情况下，数据库应当在编写任何代码前就设计好。但总会有一些情况需要你需要添加新的业务逻辑或者修改数据库配置，这时就需要用到数据库迁移了。

## Atlas

[Atlas](https://github.com/ariga/atlas) 是一个强大的数据库结构管理工具，能够以代码的形式管理数据库结构。Atlas 有两种工作流，它不仅可以像 Laravel migration 一样管理迁移，还可以直接查找并应用任何的数据库结构变更。你可以在[这里](https://atlasgo.io/getting-started)查看他的使用文档。

下面是一个在我们的 `bootcamp` 项目中引入 Atlas 版本管理工作流的示例：
{{% steps %}}

### 添加配置

在项目根目录下添加一个 `atlas.hcl` 文件：

```hcl {filename="atlas.hcl"}
env "local" {
  src = "file://manifest/database/tables/"
  dev = "docker://maria/latest/bootcamp"
  url = "maria://dbadmin:Password@localhost:3306/bootcamp"
  migration {
    dir = "file://manifest/database/migrations/"
  }
}
env "docker" {
  src = "file://manifest/database/tables/"
  dev = "docker://maria/latest/bootcamp"
  url = "maria://dbadmin:Password@database:3306/bootcamp"
  migration {
    dir = "file://manifest/database/migrations/"
  }
}
```
- **src**: 数据库结构文件目录。在我们的 `bootcamp` 项目中，我们可以在 `manifese/database/tables/` 目录下创建所有的表结构，并将该目录设为 `src`。
- **dev**: [_Dev Database_](https://atlasgo.io/atlas-schema/sql) 用于给 atlas 查找变更。
- **url**: 你的数据库链接。
- **dir**: 迁移文件目录。

### 创建迁移文件

尽管在官方文档中有一些方法可以基于现有的数据库进行迁移，通过 docker 使用一个空数据库显然更加简单。删除旧的容器和卷，运行 `docker compose up -d --build` ，然后不要初始化数据库结构。

然后运行以下命令来创建迁移文件：
```bash
atlas migrate diff initial --env local
```

你会在 `manifest/database/migrations` 目录下找到由 atlas 创建的一个 `SQL` 文件和一个 `sum` 文件。

### 应用迁移

你可以通过以下命令来应用迁移：
```bash
atlas migrate apply --env local
```

为了开发时更加方便，我们可以修改一下 dockerfile：
```dockerfile {filename="Dockerfile"}
FROM golang:1.22-alpine as builder
WORKDIR /tmp
RUN wget -O gf "https://github.com/gogf/gf/releases/latest/download/gf_$(go env GOOS)_$(go env GOARCH)" && \
    chmod +x gf && \
    ./gf install -y && \
    rm ./gf

FROM golang:1.22-alpine as dev
WORKDIR /var/www
COPY --from=builder /go/bin/gf /go/bin/gf
COPY --from=arigaio/atlas:latest-alpine /atlas /usr/local/bin/atlas
COPY manifest/database/migrations /migrations
CMD atlas migrate apply --env docker && \
    gf run main.go

ENV DEBIAN_FRONTEND noninteractive
RUN apk add --no-cache tzdata
ENV TZ=Asia/Shanghai
```

现在当容器启动时，迁移就会自动应用了。你也可以带上参数 `--env docker` 在容器中使用 atlas 的命令行工具了。

{{% /steps %}}
