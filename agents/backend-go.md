# Backend Go Agent

## 必读

1. [global.md](./global.md)
2. [../overview/project-spec.md](../overview/project-spec.md)
3. [../backend/index.md](../backend/index.md)
4. [../api/index.md](../api/index.md)
5. [../schema/index.md](../schema/index.md)
6. [../changes/migration/current-status.md](../changes/migration/current-status.md)

## 工程边界

- `handler`：HTTP 参数绑定、鉴权上下文读取、调用 service、返回 response。
- `service`：业务动作、跨 repository 编排、事务边界。
- `repository`：数据库读写、字段白名单、软删除状态过滤、索引查询口径。
- `model`：结构定义、枚举类型、字段取值校验。
- `pkg`：基础设施包，不依赖 `internal/config`。

## 当前重点

- 先让迁移后的 Go 版本跑通业务主链路。
- auth 已打通，publish 第一阶段 HMD 后端链路已通过真实服务联调。
- HMD 是当前主数据基础，HPD 是后续展示层数据。

## 修改规则

1. 不做兼容旧数据库或旧前端的逻辑。
2. 不大范围重构 Wire，除非当前链路必须接入。
3. 新增 model 时按 `domain.go / domain_enum.go / domain_validation.go` 拆。
4. 新增 repository 时按模块建目录，集合多时不要塞进一个大文件。
5. 新增 publish 逻辑时，handler 只调 `PublishService`，不要直接调 HMD 子 service 或 repository。
6. 新增字段约束时，同步 schema 和 model validation。
7. 新增接口路径必须是 `POST /api/v1/{module}/{action}`，禁止 `/module/domain/action` 这类多级业务路径。

## 验证命令

```bash
gofmt -w <changed-go-files>
go test ./...
go vet ./...
go build ./...
```
