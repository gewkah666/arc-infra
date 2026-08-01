# arc-infra

Arc 后端的"编排层"仓库。

不包含任何业务代码。包含：

- **Postgres schema**：`docker/db/init.sql`（数据库初始化）+ `migrate.sql`（手动 ALTER 补丁）
- **top-level docker-compose.yml**：把 [arc-gateway](../arc-gateway)、[arc-core](../arc-core)、[arc-web-admin](../arc-web-admin) 三个上游仓库的镜像编排成一个 stack

## 仓库结构

```
arc-infra/
├── docker-compose.yml       # 完整 stack 的编排
├── docker/
│   └── db/
│       ├── init.sql         # 容器首次启动时执行
│       └── migrate.sql      # 手动运行的 ALTER 补丁
├── .env.example             # 跨服务的环境变量示例
├── README.md
└── AGENTS.md
```

## 快速开始

```bash
# 启动基础设施（Postgres + Redis + RabbitMQ）
docker-compose up -d db redis rabbitmq

# 构建并启动所有服务（依赖 ../arc-gateway, ../arc-core, ../arc-web-admin）
docker-compose up --build

# 查看日志
docker-compose logs -f gateway
```

## 服务端口

| 服务 | 端口 | 备注 |
|---|---|---|
| gateway | 8000 | REST + SSE，依赖 pose-service |
| web-admin | 3000 | 静态 SPA，反向代理 /api 到 gateway |
| pose-service | 50051 | gRPC 内部端口 |
| db | 5432 | Postgres 16 |
| redis | 6379 | 缓存 + Celery result backend |
| rabbitmq | 5672 + 15672 | Celery broker；15672 是管理界面 |

## 默认账号

`admin` / `admin123`（在 gateway 容器首次启动时通过 `APP__ADMIN_INITIAL_USERNAME` / `APP__ADMIN_INITIAL_PASSWORD` seed）。**生产环境务必修改**。

## Schema 演进

- 表结构集中在 `docker/db/init.sql`。修改表结构时，编辑本仓库，然后 PR 到 `main`。
- arc-gateway 和 arc-core 都假设这个 schema。schema 变更需要同步更新两边的查询/模型代码。

## Build context

`docker-compose.yml` 用相对路径 `../arc-<svc>` 引用上游仓库。CI 环境必须在 `arc-infra/` 目录的同级同时存在三个上游仓库的克隆（`../arc-gateway`、`../arc-core`、`../arc-web-admin`）。

如果只想跑单个服务，可以直接进入对应上游仓库的目录跑 `docker build`。每个上游仓库有独立的 `.github/workflows/ci.yml`。

## License

Apache License 2.0