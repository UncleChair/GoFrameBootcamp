---
title: Database
type: docs
weight: 1
---

Create a `database` folder in `manifest` folder, you can store you database related code or settings here.

## Setup
{{< tabs items="Docker,Local" >}}
{{< tab >}}
Create docker-compose file in your project root folder:
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
we are using __mariadb__ as database in this file, you could also change to any other database supported by GoFrame.
{{< /callout >}}
{{< /tab >}}
{{< tab >}}
Since you are using local database, you need to do following steps by yourself:
1. Create database `bootcamp`
2. Create account for your project
    - username: dbadmin
    - password: Password
3. Add permissions to created account

{{< callout type="warning" >}}
Using root account is not recommended
{{< /callout >}}
{{< /tab >}}
{{< /tabs >}}

## Schema

Since GoFrame is an enterprise-class framework, its design principle is not allowing  developers to change schema when developing. Therefore, we need to create a database schema first. You may also find other project for database schema management:
{{< cards >}}
  {{< card link="/web-server/additional/schema" title="Schema management" icon="document-text" subtitle="Migration when developing" >}}
{{< /cards >}}

For simplicity, we will just use the following SQL to create the schema first, you may discover other methods later:
```sql {filename="manifest/database/initial.sql"}
CREATE TABLE IF NOT EXISTS `users`
(
    `uid` varchar(10) NOT NULL UNIQUE COMMENT 'User Unique ID',
    `username`  varchar(25) NOT NULL UNIQUE COMMENT 'User Name',
    `password`  varchar(45) NOT NULL COMMENT 'User Password',
    PRIMARY KEY (`uid`),
    INDEX `idx_username` (`username`),
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS `messages`
(
    `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'Message ID',
    `user_uid`  varchar(10) NOT NULL UNIQUE COMMENT 'Message Sender Uid',
    `content`  varchar(100) NOT NULL UNIQUE COMMENT 'Message Content',
    `created_at` datetime DEFAULT NULL COMMENT 'Created Time',
    `updated_at` datetime DEFAULT NULL COMMENT 'Updated Time',
    `deleted_at` datetime DEFAULT NULL COMMENT 'Soft deleted Time',
    PRIMARY KEY (`id`),
    FOREIGN KEY (`user_uid`) REFERENCES `users`(`uid`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Initialize tables

{{< tabs items="Docker,Local" >}}
{{< tab >}}
Start the database container with:
```bash
docker compose up -d --build
```
Enter the container and execute the sql file in `var/database` folder:
```bash
mysql -u dbadmin -pPassword bootcamp < /var/database/initial.sql
```
{{< /tab >}}
{{< tab >}}
Use the following command to execute the sql file:
```bash
mysql -u dbadmin -pPassword bootcamp < /var/database/initial.sql
```
{{< /tab >}}
{{< /tabs >}}
