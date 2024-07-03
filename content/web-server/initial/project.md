---
title: Project
type: docs
weight: 1
---

With GoFrame CLI tools, we could create a new project quickly and then configure the compose file for docker.

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
    args:  "all"
```
- **path**: Set output directory path for built binary file. it's `./` in default
- **args**: Custom arguments for `gf run`

{{% details title="Additional config for `gf run`" closed="true" %}}
```yaml {filename="hack/config.yaml"}
gfcli:
  run:
    path:  "bin"
    extra: ""
    args:  "all"
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
```yaml {filename="hack/config.yaml"}
gfcli:
  gen:
    dao:
    - link: "link to your database"
```
Keep it blank since we have not configure our database yet. We may deal with it later.

#### Build project
As we mentioned before, Go is a compiled language. So we need to build our project after we finish all the development. The command for this is `gf build`:
```yaml {filename="hack/config.yaml"}
gfcli:
  build:
    arch: "all"
    system: "all"
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
    extra: ""
    args:  "all"
    watchPaths:
    - api/*.go
    - internal/controller/*.go
  gen:
    dao:
    - link: ""
  build:
    arch: "all"
    system: "all"
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
  level : "all"
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
  level : "all"
  stdout: true
```

If `path` is not set, the log will only be output to stdout.

#### Database

Although we have not configured the database, we could add logger to the database first:
```yaml {filename="manifest/config/config.yaml"}
database:
  logger:
    path: "/var/log/ServerLog"
    file: "database-{Y-m-d}.log"
    level: "all"
    stdout: false
```

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
  level : "all"
  stdout: true
database:
  logger:
    path: "/var/log/ServerLog"
    file: "database-{Y-m-d}.log"
    level: "all"
    stdout: false
```
{{% /details %}}

{{< callout type="info" >}}
For simplicity, we only use part of the config settings for CLI tools and project, you could find more details here:
- [CLI tools](https://goframe.org/pages/viewpage.action?pageId=1114260)
- [Http server](https://github.com/gogf/gf/blob/master/net/ghttp/ghttp_server_config.go)
- [Logger](https://github.com/gogf/gf/blob/master/os/glog/glog_logger_config.go)
- [Database](https://github.com/gogf/gf/blob/master/database/gdb/gdb_core_config.go)
{{< /callout >}}