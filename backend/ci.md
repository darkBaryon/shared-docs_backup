# Backend CI

Go 后端 CI 当前只做基础质量门禁，目标是先保证每次 push / PR 不破坏格式、单元测试和构建。

## 默认流水线

配置文件：

```text
.github/workflows/ci.yml
```

默认执行：

```bash
gofmt -l .
go test ./...
go build ./...
```

约定：

- `go test ./...` 必须保持不依赖真实 Mongo / Redis。
- 真实 Mongo / Redis 集成测试默认跳过，由显式环境变量开启。
- 默认 CI 不注入 `.env`，也不读取本地私密配置。
- checkout 必须启用 submodule，因为接口与工程规范在 `shared-docs`。

## 集成测试阶段

后续需要把集成测试纳入 CI 时，再单独增加带 Mongo / Redis service container 的 job。

第一阶段不把集成测试塞进默认 job，避免基础 CI 被外部依赖稳定性拖慢或误伤。
