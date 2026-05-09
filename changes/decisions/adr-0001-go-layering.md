# ADR 0001: Go 后端分层

## 决策

Go 后端采用：

```text
handler -> service -> repository -> pkg/database
```

## 说明

- handler 只处理 HTTP。
- service 承载业务动作和编排。
- repository 承载数据库读写边界。
- model 承载结构、枚举和字段约束。

## 约束

Python 项目只作为业务行为参考，不作为 Go 技术架构模板。
