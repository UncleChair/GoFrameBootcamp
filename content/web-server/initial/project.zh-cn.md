---
title: 项目
type: docs
weight: 2
---

{{< callout type="info" >}}
还记得之前的数据库信息吗？我们将在这里使用它们。
- **数据库**: bootcamp
- **用户名**: dbadmin
- **密码**: Password
{{< /callout >}}

## 配置

### 命令行工具

命令行工具的配置文件位于 `hack` 文件夹下。这里已经有了 `gf docker` 的初始配置，我们可以添加更多。这份文件可以用于添加 `gf` 相关命令的默认配置。在[这里](https://goframe.org/pages/viewpage.action?pageId=1114260)可以找到所有命令配置的细节。

#### 热重载

Go 是一门编译型语言，所以在更新任何代码后，我们都需要使用热重载来重新编译代码。这里有一个方便的指令 `gf run`，将它的设置添加到 `gfcli` 下：
```yaml {filename="hack/config.yaml"}
gfcli:
  run:
    path:  "bin"
    args:  ""
```
- **path**: 设置二进制文件的输出路径，默认为 `./`
- **args**: `gf run` 的参数

{{% details title=" `gf run` 的更多配置" closed="true" %}}
```yaml {filename="hack/config.yaml"}
gfcli:
  run:
    path:  "bin"
    extra: ""
    args:  ""
    watchPaths:
    - api/*.go
    - internal/controller/*.go
```
- **extra**: 与"go run"/"go build"相同的选项，除了以下定义的一些选项
- **watchPaths**: 热重载的监听路径
{{% /details %}}

#### 代码生成
我们将会在bootcamp中使用以下代码生成指令：
- `gf gen ctrl`: 生成 controller
- `gf gen service`: 生成 service 接口
- `gf gen dao`: 生成数据库相关的代码

通常我们只需要添加 `dao` 相关的配置，因为它需要数据库的链接。

{{< tabs items="MariaDB" >}}
{{< tab >}}
```yaml {filename="hack/config.yaml"}
gfcli:
  gen:
    dao:
    - link: "mariadb:dbadmin:Password@tcp(database:3306)/bootcamp"
```
{{< /tab >}}
{{< /tabs >}}
{{< callout type="info" >}}
如果在容器中使用 `gf` 指令，host 应当为 `database` 而不是 `localhost`。如果想在容器外使用 `gf gen dao` 你应当将 host 设置为 `localhost`。
{{< /callout >}}

数据库链接的格式为：
```bash
类型:账号:密码@协议(地址)/数据库名称?特性配置
```
你可以使用类似的方法连接其他[受支持的数据库](https://github.com/gogf/gf/tree/master/contrib/drivers)。

#### 编译项目
我们提到过 Go 是一门编译型语言。因此我们需要在开发完成后编译我们的项目。相关的指令为 `gf build`：
```yaml {filename="hack/config.yaml"}
gfcli:
  build:
    arch: "amd64"
    system: "linux"
    output: "./tmp/bootcamp"
    dumpEnv: true
```
- **arch**: 设置目标架构
- **system**: 设置目标系统
- **output**: 设置二进制文件输出路径
- **dumpEnv**: 编译时显示环境变量

{{% details title="命令行完整配置" closed="true" %}}
```yaml {filename="hack/config.yaml"}
gfcli:
  docker:
    build: "-a amd64 -s linux -p temp -ew"
    tagPrefixes:
      - my.image.pub/my-app
  run:
    path:  "bin"
    args:  ""
  gen:
    dao:
    - link: "mariadb:dbadmin:Password@tcp(database:3306)/bootcamp"
  build:
    arch: "amd64"
    system: "linux"
    output: "./tmp/bootcamp"
    dumpEnv: true
```
{{% /details %}}

### 项目

与命令行工具配置类似，项目在 `manifest/config` 文件夹下也有默认的配置文件。这里面包含了一些项目的初始设置:
```yaml {filename="manifest/config/config.yaml"}
server:
  address:     ":8000"
  openapiPath: "/api.json"
  swaggerPath: "/swagger"

logger:
  level: "all"
  stdout: true
```
我们需要修改 `server` 和 `logger` 的配置，并添加 `database` 配置。

#### 服务

现在，配置文件已经声明了用于设置监听地址的 `address` 属性。你可以将它更改为 `:8080` 或者其他没有冲突的端口。

`openapiPath` 和 `swaggerPath` 属性分别用于设置 OpenAPI 和 Swagger 文档的 URL。你可以通过它们来查看自动生成的API文档。

除此之外，我们还可以添加其他的server配置：
```yaml {filename="manifest/config/config.yaml"}
server:
  address: ":8000"
  openapiPath: "/api.json"
  swaggerPath: "/swagger"
  graceful: true
  gracefulTimeout: 10
  errorLogEnabled: true
  errorLogPattern: "error-{Y-m-d}.log"
```
- **graceful**: 是否启用平滑功能
- **gracefulTimeout**: 平滑功能的超时时间
- **errorLogEnabled**: 是否开启错误日志
- **errorLogPattern**: 错误日志文件名格式

#### 日志

我们可以为 `logger` 添加日志输出目录：

```yaml {filename="manifest/config/config.yaml"}
logger:
  path: "/var/log/ServerLog"
  level: "all"
  stdout: true
```

如果没有设置 `path`，日志将只会输出到标准输出（命令行）。

#### 数据库

在添加数据库连接之前，我们可以先添加一个用于debug的数据库日志：
```yaml {filename="manifest/config/config.yaml"}
database:
  logger:
    path: "/var/log/ServerLog"
    file: "database-{Y-m-d}.log"
    level: "all"
    stdout: false
```
然后我们可以添加默认的数据库配置：
```yaml {filename="manifest/config/config.yaml"}
database:
  default:
    host: "database"
    port: "3306"
    user: "dbadmin"
    pass: "Password"
    name: "bootcamp"
    type: "mariadb"
    timezone: "Asia/Shanghai"
    extra: "parseTime=true"
    role: "master"
    debug: "true"
    dryrun: 0
    charset: "utf8mb4"
    maxLifetime: "10s"
```

{{< callout type="info" >}}
由于应用将在容器中运行，我们需要使用 `database` 而不是 `localhost` 作为 host。
{{< /callout >}}

{{% details title="项目完整配置" closed="true" %}}
```yaml {filename="manifest/config/config.yaml"}
server:
  address: ":8000"
  openapiPath: "/api.json"
  swaggerPath: "/swagger"
  graceful: true
  gracefulTimeout: 10
  errorLogEnabled: true
  errorLogPattern: "error-{Y-m-d}.log"
logger:
  path: "/var/log/ServerLog"
  level: "all"
  stdout: true
database:
  logger:
    path: "/var/log/ServerLog"
    file: "database-{Y-m-d}.log"
    level: "all"
    stdout: false
  default:
    host: "database"
    port: "3306"
    user: "dbadmin"
    pass: "Password"
    name: "bootcamp"
    type: "mariadb"
    timezone: "Asia/Shanghai"
    extra: "parseTime=true"
    role: "master"
    debug: "true"
    dryrun: 0
    charset: "utf8mb4"
    maxLifetime: "10s"
```
{{% /details %}}

{{< callout emoji="🌐" >}}
为保简洁，我们只使用了命令行和项目的部分配置，这里可以找到更多配置细节：
- [命令行工具](https://goframe.org/pages/viewpage.action?pageId=1114260)
- [服务](https://github.com/gogf/gf/blob/master/net/ghttp/ghttp_server_config.go)
- [日志](https://github.com/gogf/gf/blob/master/os/glog/glog_logger_config.go)
- [数据库](https://github.com/gogf/gf/blob/master/database/gdb/gdb_core_config.go)
{{< /callout >}}

## 使用 Dokcer 运行

{{< callout type="warning" >}}
- 在接下来的步骤中，我们会将本地文件夹挂载至容器内。由于 [WSL](https://github.com/microsoft/WSL/issues/4739) 的一些问题，某些情况下热重载将不会被触发。请确保使用 Windows 时在 WSL 下修改代码，或者更简单一些，将所有文件放在 WSL 下，然后通过你的 IDE 连接并开发。
- 你也可以自行编写配置在本地使用热重载功能，并只将编译后的文件放入容器中。
{{< /callout >}}

在所有配置完毕后，我们可以开始配置容器了。

在项目根目录下创建一个 `Dockerfile` 文件：
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
CMD gf run main.go
ENV DEBIAN_FRONTEND noninteractive
RUN apk add --no-cache tzdata
ENV TZ=Asia/Shanghai
```
{{< callout type="warning" >}}
go apline 镜像缺少一些时区相关的包，请在 Dockerfile 的最后加上这些配置。
{{< /callout >}}

然后将以下配置添加到 `docker-compose.yaml` 文件中的 `services` 下：
```yaml {filename="docker-compose.yaml"}
web:
  build:
    dockerfile: Dockerfile
    context: .
    target: dev
  depends_on:
    database:
      condition: service_healthy
  volumes:
    - ./:/var/www
    - ./storage:/var/www/storage
  environment: &bootcamp-environment
    CONTAINER_ROLE: web
    DB_HOST: database
    DB_PORT: 3306
    DB_DATABASE: bootcamp
    DB_USERNAME: dbadmin
    DB_ROOT_PASSWORD: R0OTp@ssword
    DB_PASSWORD: Password
    DB_DRIVER: mysql
  ports:
    - "8000:8000"
```

{{% details title="完整的 `docker-compose.yaml` 文件" closed="true" %}}
```yaml {filename="docker-compose.yaml"}
version: "3"
services:
  database:
    image: mariadb:10.3
    environment:
      MYSQL_ROOT_PASSWORD: R0OTp@ssword
      MYSQL_DATABASE: bootcamp
      MYSQL_USER: dbadmin
      MYSQL_PASSWORD: Password
      TZ: Asia/Shanghai
    healthcheck:
      test: mysqladmin ping -u dbadmin -pPassword
    volumes:
      - ./manifest/database/:/var/database
      - database:/var/lib/mysql
    command:
      mysqld --default-time-zone=Asia/Shanghai --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    ports:
      - "3306:3306"
  web:
    build:
      dockerfile: Dockerfile
      context: .
      target: dev
    depends_on:
      database:
        condition: service_healthy
    volumes:
      - ./:/var/www
      - ./storage:/var/www/storage
    environment: &bootcamp-environment
      CONTAINER_ROLE: web
      DB_HOST: database
      DB_PORT: 3306
      DB_DATABASE: bootcamp
      DB_USERNAME: dbadmin
      DB_ROOT_PASSWORD: R0OTp@ssword
      DB_PASSWORD: Password
      DB_DRIVER: mysql
    ports:
      - "8000:8000"
volumes:
  database:
```
{{% /details %}}

现在再次使用 `docker-compose up -d --build` 来编译并启动所有容器。等待一段时间，你会看到项目在运行中。

查看 `GoFrame` 自动生成的 [hello](http://localhost:8000/hello) 页面和 [Swagger](http://localhost:8000/swagger) 页面。

{{< callout type="info" >}}
因为我们将本地文件夹挂载到了容器中，有时候 Windows Defender 会将 `gf` 作为病毒隔离，你可以直接允许它运行。
{{< /callout >}}