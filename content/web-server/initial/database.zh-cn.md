---
title: 数据库
type: docs
weight: 1
---

在 `manifest` 文件夹下创建一个 `database` 文件夹，你可以在这里存储和数据库相关代码或设置。

## 设置
{{< tabs items="Docker,Local" >}}
{{< tab >}}
在项目根目录下创建 docker-compose.yaml 文件：
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
volumes:
  database:
```
{{< callout type="info" >}}
在本文件中，我们使用 __mariadb__ 数据库，你也可以将它改为 GoFrame 支持的任意数据库。
{{< /callout >}}
{{< /tab >}}
{{< tab >}}
由于你正在使用本地数据库，你需要自行完成以下的步骤：
1. 创建数据库 `bootcamp`
2. 为项目创建数据库账号
    - 用户名：dbadmin
    - 密码：Password
3. 添加数据库账号权限

{{< callout type="warning" >}}
我们不推荐使用 root 用户
{{< /callout >}}
{{< /tab >}}
{{< /tabs >}}

## 数据结构

由于 GoFrame 是一款企业级的框架，它的设计原则就是不允许开发者在开发时自行修改数据库结构。因此我们需要先确定数据库结构。你也可以找到其他用于管理数据库结构的项目:
{{< cards >}}
  {{< card link="/web-server/additional/schema" title="数据库结构管理" icon="document-text" subtitle="开发时的数据库迁移" >}}
{{< /cards >}}

为保简洁，我们将使用以下 SQL 创建数据库结构，你可以稍后自行探索其他方法：
```sql {filename="manifest/database/initial.sql"}
CREATE TABLE IF NOT EXISTS `users`
(
    `uid` varchar(10) NOT NULL UNIQUE COMMENT 'User Unique ID',
    `username`  varchar(25) NOT NULL UNIQUE COMMENT 'User Name',
    `password`  varchar(45) NOT NULL COMMENT 'User Password',
    PRIMARY KEY (`uid`),
    INDEX `idx_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS `messages`
(
    `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'Message ID',
    `user_uid`  varchar(10) NOT NULL COMMENT 'Message Sender Uid',
    `content`  varchar(100) NOT NULL COMMENT 'Message Content',
    `created_at` datetime DEFAULT NULL COMMENT 'Created Time',
    `updated_at` datetime DEFAULT NULL COMMENT 'Updated Time',
    `deleted_at` datetime DEFAULT NULL COMMENT 'Soft deleted Time',
    PRIMARY KEY (`id`),
    FOREIGN KEY (`user_uid`) REFERENCES `users`(`uid`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 初始化

{{< tabs items="Docker,Local" >}}
{{< tab >}}
使用以下指令启动数据库容器：
```bash
docker compose up -d --build
```
进入容器并执行 `var/database` 下的 sql 文件:
```bash
mysql -u dbadmin -pPassword bootcamp < /var/database/initial.sql
```
{{< /tab >}}
{{< tab >}}
使用以下指令执行 sql 文件：
```bash
mysql -u dbadmin -pPassword bootcamp < /var/database/initial.sql
```
{{< /tab >}}
{{< /tabs >}}

### 插入数据

进入容器并执行以下指令来进入你的数据库：

```bash
mysql -u dbadmin -pPassword bootcamp
```

使用以下指令将数据插入 `users` 表：

```sql
INSERT IGNORE INTO `users`(`uid`, `username`, `password`)
VALUES ('0000000000', 'admin', '123456');
```

检查数据：

```sql
SELECT * FROM `users`;
```

| uid         | username    | password    |
| ----------- | ----------- | ----------- |
| 0000000000  | admin       | 123456      |

{{< callout type="info" >}}
- 不推荐储存明文密码，你可以稍后添加你的加密算法。
- 本条记录是删增改查部分需要的，因为我们在 `messages` 表中添加了 `FOREIGN KEY`。
{{< /callout>}}