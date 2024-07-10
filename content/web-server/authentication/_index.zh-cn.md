---
title: 用户认证
weight: 3
---

用户认证一直是网络应用中的一个重要组成部分，你永远不会希望用户能够不受控制的访问所有资源。因此，我们首先要在项目中添加用户认证功能。

认证用户的方法有很多，你可以在下面的列表中找一种你最喜欢的方案并学习如何实现它。如果你不确定该选择哪一个，我们推荐你先试试 **Json Web Token**(JWT)。增删改查部分默认使用的认证方式也是 JWT。

{{< cards >}}
  {{< card link="jwt" title="Json Web Token" icon="document-text" subtitle="低占用、无状态" >}}
  <!-- {{< card link="session" title="Session" icon="document-text" subtitle="Built-in with GoFrame" >}} -->
{{< /cards >}}

