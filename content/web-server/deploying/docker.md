---
title: Docker
type: docs
weight: 2
---

Deploy your application with docker is also a good choice. You may not need to worry about dependence problem or environment settings.

We have used docker for our development, you could have a try with those images.

{{< callout type="warning" >}}
Do not use your development docker image directly for real production, it is not safe.
{{< /callout >}}

Most of the time we would use `gf build` to generate the final binary file and use other command to package resources. Then we put those files into a docker image. Delver it to the production server and we could use it directly.

Here is an example docker file for building production image:
```dockerfile
FROM loads/alpine

LABEL maintainer="admin@bootcamp.com"

ENV WORKDIR /app/main

# Add your binary file and give permission
ADD ./temp/bootcamp $WORKDIR/bootcamp
RUN chmod +x $WORKDIR/bootcamp

# Add static resources
ADD resource $WORKDIR/resource

WORKDIR $WORKDIR
CMD ./bootcamp
```