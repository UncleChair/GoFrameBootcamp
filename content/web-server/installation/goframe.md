---
title: GoFrame CLI
type: docs
weight: 2
---

GoFrame CLI provides useful command to help you generate code and accelerate your development.

### Install from source code

```bash
git clone https://github.com/gogf/gf && cd gf/cmd/gf && go install
```

### Install from release

{{< tabs items="Linux,MacOS,Windows" >}}
{{< tab >}}
```bash
wget -O gf "https://github.com/gogf/gf/releases/latest/download/gf_$(go env GOOS)_$(go env GOARCH)" && chmod +x gf && ./gf install -y && rm ./gf
```

{{< callout type="warning" >}}
You may have following problems:
- Install `wget` if it is not installed.
- If you are using `zsh`, you may meet conflict with `gf` command. Use `alias gf="gf"` to solve it. If it still not work, try add `gf` bin file to `/usr/local/bin`.
{{< /callout >}}
{{< /tab >}}
{{< tab >}}
```bash
wget -O gf "https://github.com/gogf/gf/releases/latest/download/gf_$(go env GOOS)_$(go env GOARCH)" && chmod +x gf && ./gf install -y && rm ./gf
```

{{< callout type="warning" >}}
You may have following problems:
- Install `wget` if it is not installed.
- If you are using `zsh`, you may meet conflict with `gf` command. Use `alias gf="gf"` to solve it. If it still not work, try add `gf` bin file to `/usr/local/bin`.
{{< /callout >}}
{{< /tab >}}
{{< tab >}}
Download the release version [here](https://github.com/gogf/gf/releases) and double click to install. 
{{< /tab >}}
{{< /tabs >}}

### Check your installation

Use `gf -v` to check your installation.
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