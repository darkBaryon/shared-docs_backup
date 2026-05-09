# Backend Config

## 约定

- `internal/config` 负责读取 YAML、`.env` 和环境变量。
- `pkg/database/mongo`、`pkg/database/redis` 自己定义 Config。
- `internal/config` 负责把项目配置转换为 pkg Config。
- 敏感信息不写死在 YAML，优先从 `.env` 或环境变量读取。
- timeout 在 YAML 中使用 `"10s"` 这种字符串，转换到 pkg Config 时使用 `time.Duration`。
