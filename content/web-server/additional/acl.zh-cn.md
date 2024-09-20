---
title: 访问控制
type: docs
weight: 3
---

在验证用户并授予其访问权限后，我们仍需要一种机制来控制用户的行为。最经典的做法是使用允许或禁止用户访问特定资源的规则表，也就是我们常说的 ACL (Access Control List)。不过自行实现 ACL 需要花费不少时间，因此使用成熟的开源项目也是一个不错的选择。

## Casbin

`Casbin` 是一个强大且高效的开源访问控制库，支持各种访问控制模型，用于在全局范围内执行授权。它不仅功能强大，也支持多种语言，你可以在[这里](https://casbin.org/zh/docs/get-started)找到它的具体使用方法。

[适配器列表](https://casbin.org/zh/docs/adapters)中也有支持 `GoFrame` 的适配器。