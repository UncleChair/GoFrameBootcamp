---
title: Database Schema Control
type: docs
weight: 2
---

In most cases, database should be designed and created before writing any code. But since there would always be a chance that you may need to add new business logic or modify database configurations, migrations for database would be needed at that time.

## Atlas

[Atlas](https://github.com/ariga/atlas) is a powerful database schema management tool to manage schema as code. It has two workflows that could not only manage migrations in traditional ways like Laravel migration, but could also find and apply changes to database overall schemas. You may find usage documentation [here](https://atlasgo.io/getting-started). 

Here is an example to introduce versioned workflow with atlas to our `bootcamp` project:

{{% steps %}}

### Add configurations

Create a `atlas.hcl` file in the root directory:
```hcl {filename="atlas.hcl"}
env "local" {
  src = "file://manifest/database/tables/"
  dev = "docker://maria/latest/bootcamp"
  url = "maria://dbadmin:Password@localhost:3306/bootcamp"
  migration {
    dir = "file://manifest/database/migrations/"
  }
}
env "docker" {
  src = "file://manifest/database/tables/"
  dev = "docker://maria/latest/bootcamp"
  url = "maria://dbadmin:Password@database:3306/bootcamp"
  migration {
    dir = "file://manifest/database/migrations/"
  }
}
```
- **src**: The database schema files directory, in our `bootcamp` project, we could create all tables in `manifese/database/tables/` folder and use it as `src`.
- **dev**: [_Dev Database_](https://atlasgo.io/atlas-schema/sql) is used to compute the diff by atlas.
- **url**: Your database URL.
- **dir**: The migration files directory.

### Create migrations

There are some methods to use migrations with existing database in official documentation, but it would be much simple to use an empty database with docker. Just delete the old container and volume created before and run `docker compose up -d --build` without initial database tables.

Then run the following command to create migrations:
```bash
atlas migrate diff initial --env local
```

You will find a `SQL` file and a `sum` file created by atlas in `manifest/database/migrations` folder.

### Apply migration

You can simple apply the initial database schema with:
```bash
atlas migrate apply --env local
```

To make it more convenient, we could modify our dockerfile:
```dockerfile {filename="Dockerfile"}
FROM golang:1.22-alpine as builder
WORKDIR /tmp
RUN wget -O gf "https://github.com/gogf/gf/releases/latest/download/gf_$(go env GOOS)_$(go env GOARCH)" && \
    chmod +x gf && \
    ./gf install -y && \
    rm ./gf

FROM golang:1.22-alpine as dev
WORKDIR /var/www
COPY --from=builder /go/bin/gf /go/bin/gf
COPY --from=arigaio/atlas:latest-alpine /atlas /usr/local/bin/atlas
COPY manifest/database/migrations /migrations
CMD atlas migrate apply --env docker && \
    gf run main.go

ENV DEBIAN_FRONTEND noninteractive
RUN apk add --no-cache tzdata
ENV TZ=Asia/Shanghai
```

Now migrations would be auto-applied when the container starts, and you can use atlas command tool in you container with `--env docker` argument.

{{% /steps %}}
