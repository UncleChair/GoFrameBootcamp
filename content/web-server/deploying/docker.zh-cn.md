---
title: Docker
type: docs
weight: 2
---

通过 docker 部署应用也是一个好选择。这样你就不需要再担心依赖问题或者环境设置了。

我们已经在开发过程中使用过 docker了，你可以试试那些镜像。

{{< callout type="warning" >}}
不要在实际生产中使用你的开发镜像，这样不安全。
{{< /callout >}}

大部分时候我们会使用 `gf build` 生成最终的二进制文件，然后使用其他命令来打包资源，最后将它们放入一个 docker 镜像中。把这个镜像发送到生产服务器上，我们就可以直接使用它了。

下面是一个构建生产环境 docker 镜像的示例文件：
```dockerfile
FROM loads/alpine

LABEL maintainer="admin@bootcamp.com"

ENV WORKDIR /app/main

# 添加二进制文件并设置执行权限
ADD ./temp/bootcamp $WORKDIR/bootcamp
RUN chmod +x $WORKDIR/bootcamp

# 添加其他资源
ADD resource $WORKDIR/resource

WORKDIR $WORKDIR
CMD ./bootcamp
```