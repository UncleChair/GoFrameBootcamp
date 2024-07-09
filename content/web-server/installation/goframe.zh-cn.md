---
title: GoFrame 命令行工具
type: docs
weight: 2
---

GoFrame 命令行工具提供了一系列便捷的代码生成工具来提高开发速度。

### 源码安装

```bash
git clone https://github.com/gogf/gf && cd gf/cmd/gf && go install
```

### 预编译安装

{{< tabs items="Linux,MacOS,Windows" >}}
{{< tab >}}
```bash
wget -O gf "https://github.com/gogf/gf/releases/latest/download/gf_$(go env GOOS)_$(go env GOARCH)" && chmod +x gf && ./gf install -y && rm ./gf
```

{{< callout type="warning" >}}
你可能会遇到以下问题
- 安装 `wget` 如果无法使用该指令
- 如果正在使用 `zsh`，你可能会遇到 `gf` 指令冲突的问题。使用 `alias gf="gf"` 来解决。如果仍然无法成功，尝试将 `gf` 二进制文件移动至 `/usr/local/bin` 目录下。
{{< /callout >}}
{{< /tab >}}
{{< tab >}}
```bash
wget -O gf "https://github.com/gogf/gf/releases/latest/download/gf_$(go env GOOS)_$(go env GOARCH)" && chmod +x gf && ./gf install -y && rm ./gf
```

{{< callout type="warning" >}}
你可能会遇到以下问题
- 安装 `wget` 如果无法使用该指令
- 如果正在使用 `zsh`，你可能会遇到 `gf` 指令冲突的问题。使用 `alias gf="gf"` 来解决。如果仍然无法成功，尝试将 `gf` 二进制文件移动至 `/usr/local/bin` 目录下。
{{< /callout >}}
{{< /tab >}}
{{< tab >}}
在[这里](https://github.com/gogf/gf/releases)下载预编译版本并双击安装。
{{< /tab >}}
{{< /tabs >}}

### 检查安装

使用 `gf -v` 来检查安装情况。
```bash
$ gf -v
v2.7.2
Welcome to GoFrame!
Env Detail:
  Go Version: go1.22.2 linux/amd64
  GF Version(go.mod): cannot find go.mod
CLI Detail:
  Installed At: /usr/local/bin/gf
  ...
```