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

Go is a compiled language, so we need to use hot reload after we change any code. There is a useful command called `gf run`, add this command settings to `hack/config.yaml` bellow `gfcli`:
```yaml
gfcli:
  run:
    path:  "bin"
    args:  "all"
```
- **path**: Set output directory path for built binary file. it's `./` in default
- **args**: Custom arguments for `gf run`

{{% details title="Additional config for `gf run`" closed="true" %}}
```yaml
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
Typically, we only need to add `dao` related code config to `hack/config.yaml`, since it requires link to the database.
```yaml
gfcli:
  gen:
    dao:
    - link: "link to your database"
```
Keep it blank since we have not configure our database yet. We may deal with it later.

#### Build project
As we mentioned before, Go is a compiled language. So we need to build our project after we finish all the development. The command for this is `gf build`, we could also add some config to `hack/config.yaml`:
```yaml
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


{{% details title="Full config file after all settings" closed="true" %}}
```yaml
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

{{< callout type="info" >}}
For simplicity, we only use part of the config settings for CLI tools, you could find more details in [official documentation](https://goframe.org/pages/viewpage.action?pageId=1114260).
{{< /callout >}}

### Project
