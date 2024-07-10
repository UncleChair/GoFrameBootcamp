---
title: 守护进程
type: docs
weight: 1
---

Go 开发的应用可以通过守护进程的方式部署。这也是部署简单的 API 服务器时常用的方式。接下来的例子中，我们使用了 Ubuntu 系统下的 `systemctl`。
{{% steps %}}

### 设置服务器环境

在部署应用前，你需要设置服务器的环境，比如初始化数据库或者添加权限。

### 创建项目文件夹

创建一个存储可执行文件的文件夹，这里我们将在 `/home/ubuntu/` 中创建。

{{< filetree/container >}}
  {{< filetree/folder name="server" >}}
    {{< filetree/file name="api-server" >}}
    {{< filetree/folder name="static" state="closed" >}}
    {{< /filetree/folder >}}
  {{< filetree/file name="config.yaml" >}}
  {{< /filetree/folder >}}
{{< /filetree/container >}}

{{< callout type="info" >}}
记得在文件夹中添加你的配置文件或静态资源，否则将会报错。
{{< /callout >}}

### 创建服务

在 `/etc/systemd/system/` 中创建一个 service 文件。

```service {filename="/etc/systemd/system/my-api.service"}
[Unit]
Description=SERVER API
# 更改为你使用的数据库
After=mariadb.service

[Service]
Type=simple
WorkingDirectory=/home/ubuntu/server/
ExecStart=/home/ubuntu/server/api-server
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### 启动服务

通过以下命令启动服务：

```bash
sudo systemctl start my-api.service
```

{{% /steps %}}

`GoFrame` 在 `*nix` 系统下提供了平滑重启和关闭的功能，你可以使用以下命令来平滑地重启或者关闭你的服务：

- 重启服务：
```bash
kill -SIGUSR1 ProcessID
```

- 关闭服务：
```bash
kill -SIGTERM ProcessID
```

{{< callout type="info" >}}
也可以使用类似 [`supervisor`](https://pypi.org/project/supervisor/) 的其他工具来管理你的服务进程。
{{< /callout >}}
