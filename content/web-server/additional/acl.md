---
title: Access Control
type: docs
weight: 3
---

After authenticating users and granting them access, we still need a mechanism to control their behavior. The most classic approach is to use a rule table that allows or prohibits users from accessing specific resources, aka ACL (Access Control List). However, implementing ACL manually requires a lot of time, so using a mature open-sourced project is also a good choice.

## Casbin

`Casbin` is a powerful and efficient open-source access control library that supports various access control models for enforcing authorization across the board. It is not only powerful but also supports multiple programming languages. You can find its usage [here](https://casbin.org/docs/get-started).

The [adapter list](https://casbin.org/docs/adapters) also offers adapters that support `GoFrame`.