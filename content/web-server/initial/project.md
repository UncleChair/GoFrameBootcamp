---
title: Project
type: docs
weight: 2
---

With GoFrame CLI tools, we could create a new project quickly and then configure the compose file for docker.

## Reminder

{{< callout type="info" >}}
Remember our database info in the previous part? We will use them here.
- **Database**: bootcamp
- **Username**: dbadmin
- **Password**: Password
{{< /callout >}}

## Create project

Use the following command to create your project:

```bash
gf init bootcamp -u
```

Or change the `bootcamp` to whatever you want for your project name.

{{< callout type="info" >}}
GoFrame project have its own folder structure, we may talk about some methodology later.
{{< /callout >}}

## Configurations

### CLI tool

Configuration file for CLI tool is located in `hack` folder. There is a config for `gf docker` as initial, we could add more to it. This file is used to add some default settings for `gf` related commands. Details for all commands settings could be found [here](https://goframe.org/pages/viewpage.action?pageId=1114260).

#### Hot reload

Go is a compiled language, so we need to use hot reload after we change any code. There is a useful command called `gf run`, add this command settings bellow `gfcli`:
```yaml {filename="hack/config.yaml"}
gfcli:
  run:
    path:  "bin"
    args:  ""
```
- **path**: Set output directory path for built binary file. it's `./` in default
- **args**: Custom arguments for `gf run`

{{% details title="Additional config for `gf run`" closed="true" %}}
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
- **extra**: The same options as "go run"/"go build" except some options as follows defined
- **watchPaths**: Watch paths for hot reload
{{% /details %}}

#### Code generation
There are several command for code generation that would be used in our bootcamp, they are:
- `gf gen ctrl`: Generate controller
- `gf gen service`: Generate service interface
- `gf gen dao`: Generate database related code

Typically, we only need to add `dao` related code configurations, since it requires link to the database.

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
Since we may use `gf` in container, host would be `database` rather than `localhost`. Or if you want to use `gf gen dao` out of the container, you could keep it to `localhost`.
{{< /callout >}}

The format for the link is:
```bash
type:username:password@protocol(address)[/dbname][?param1=value1&...&paramN=valueN]
```
You could use other [supported database](https://github.com/gogf/gf/tree/master/contrib/drivers) similar to this link.

#### Build project
As we mentioned before, Go is a compiled language. So we need to build our project after we finish all the development. The command for this is `gf build`:
```yaml {filename="hack/config.yaml"}
gfcli:
  build:
    arch: "amd64"
    system: "linux"
    output: "./tmp/bootcamp"
    dumpEnv: true
```
- **arch**: Set the target architecture
- **system**: Set the target system
- **output**: Set the output path for built binary file
- **dumpEnv**: Display environment variables when building

{{% details title="Full config for CLI" closed="true" %}}
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

### Project

Similar to CLI tool configuration, our project also have its own configuration file in `manifest/config` folder by default. It contains some initial settings for our application:
```yaml {filename="manifest/config/config.yaml"}
server:
  address:     ":8000"
  openapiPath: "/api.json"
  swaggerPath: "/swagger"

logger:
  level: "all"
  stdout: true
```
We still need to modify `server` and `logger` configurations and add `database` configuration.

#### Server
For now, the configuration file has declared the `address`, which is used to set listening port. You could change it to `:8080` or any other port with no conflict.

The `openapiPath` and `swaggerPath` are used to set the URI of OpenAPI and Swagger, you may find it useful to check the auto-generated API documentation.

Besides these, we could still add some configurations for the server:
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
- **graceful**: Whether to enable graceful functions.
- **gracefulTimeout**: The timeout for request using graceful functions.
- **errorLogEnabled**: Whether to enable error log, default to `true`.
- **errorLogPattern**: The pattern of error log.

#### Logger

For `logger`, we could add a path to output log file:

```yaml {filename="manifest/config/config.yaml"}
logger:
  path: "/var/log/ServerLog"
  level: "all"
  stdout: true
```

If `path` is not set, the log will only be output to stdout.

#### Database

Before we add connection to database, a logger would be good for debugging:
```yaml {filename="manifest/config/config.yaml"}
database:
  logger:
    path: "/var/log/ServerLog"
    file: "database-{Y-m-d}.log"
    level: "all"
    stdout: false
```
Then we could add default database configurations:
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
Again, since the application will run in the container, we need to use `database` rather than `localhost` as the host.
{{< /callout >}}

{{% details title="Full config for project" closed="true" %}}
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
For simplicity, we only use part of the config settings for CLI tools and project, you could find more details here:
- [CLI tools](https://goframe.org/pages/viewpage.action?pageId=1114260)
- [Http server](https://github.com/gogf/gf/blob/master/net/ghttp/ghttp_server_config.go)
- [Logger](https://github.com/gogf/gf/blob/master/os/glog/glog_logger_config.go)
- [Database](https://github.com/gogf/gf/blob/master/database/gdb/gdb_core_config.go)
{{< /callout >}}

## Run with docker

{{< callout type="warning" >}}
- In the following steps, we are mounting folders from our local machine to the container, there are some case that the hot reload would not be triggered due to the [WSL problem](https://github.com/microsoft/WSL/issues/4739). Make sure to modify code under WSL when you are using Windows, or to be simple, just put all you need into WSL and use your IDE to connect to it.
- Or you could write your own config to only use built file in the container. Which means the hot reload would be finished by your local machine.
{{< /callout >}}

After finishing all the configurations, finally we could start setting the container.

Create a `Dockerfile` in your project folder:
```dockerfile {filename="Dockerfile"}
FROM golang:1.22-alpine as base
FROM base as dev
WORKDIR /var/www

CMD wget -O gf "https://github.com/gogf/gf/releases/latest/download/gf_$(go env GOOS)_$(go env GOARCH)" && \
    chmod +x gf && \
    ./gf install -y && \
    rm ./gf && \
    gf run main.go

ENV DEBIAN_FRONTEND noninteractive
RUN apk add --no-cache tzdata
ENV TZ=Asia/Shanghai
```
{{< callout type="warning" >}}
The go apline image is missing timezone related packages, remember to set them at the end at the Dockerfile.
{{< /callout >}}

Then add the following settings to your `services` in `docker-compose.yaml` file:
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

{{% details title="Full `docker-compose.yaml` file" closed="true" %}}
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

Now use `docker-compose up -d --build` again to start all the containers and wait for a moment, you will see the project is running now.

Check the [hello page](http://localhost:8000/hello) and [Swagger page](http://localhost:8000/swagger) auto-generated by `GoFrame`

{{< callout type="info" >}}
Sometime Windows Defender will treat `gf` as a virus since we are mounting folders to our local machine, you can just allow it.
{{< /callout >}}