# Auth Flow

认证流程详见 [../api/auth-flow.md](../api/auth-flow.md)。

## 摘要

```text
客户端
  -> auth login/register
    -> Go auth service
      -> MongoDB user/auth/profile
      -> Redis session
  -> Authorization: Bearer <token>
    -> middleware
      -> handler
```
