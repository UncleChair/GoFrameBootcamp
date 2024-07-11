---
title: Daemon
type: docs
weight: 1
prev: false
next: web-server/deploying/nginx
---

Application developed by go could be deployed with daemon mode. This is a common way when the application is a simple API server. The example here is using `systemctl` under Ubuntu.

{{% steps %}}

### Set server environment

To use your application, first you need to set your server's environment, like initialing database or adding permissions.

### Create project folder

Create a folder to store your executable file. We are creating it under `/home/ubuntu/` here.

{{< filetree/container >}}
  {{< filetree/folder name="server" >}}
    {{< filetree/file name="api-server" >}}
    {{< filetree/folder name="static" state="closed" >}}
    {{< /filetree/folder >}}
  {{< filetree/file name="config.yaml" >}}
  {{< /filetree/folder >}}
{{< /filetree/container >}}

{{< callout type="info" >}}
Remember to add your config file or static resources in the folder, or there will be some errors.
{{< /callout >}}

### Create service

Add a service file to your `/etc/systemd/system/` folder.

```service {filename="/etc/systemd/system/my-api.service"}
[Unit]
Description=SERVER API
# Change it to your database if needed
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

### Start service

Start your service with:

```bash
sudo systemctl start my-api.service
```

{{% /steps %}}

`GoFrame` provide graceful restart and shutdown functions under `*nix` system. You could use the following command to restart or shutdown your service gracefully:

- Restart service:
```bash
kill -SIGUSR1 ProcessID
```

- Shutdown service:
```bash
kill -SIGTERM ProcessID
```

{{< callout type="info" >}}
You could also use other tools like [`supervisor`](https://pypi.org/project/supervisor/) to manage your services.
{{< /callout >}}
