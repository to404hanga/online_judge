# OnlineJudge Docker 部署指南

本项目提供了完整的Docker化部署方案，包括网关服务、控制台服务以及所需的依赖服务（MySQL、Redis、MinIO）。

## 项目架构

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│                 │    │                 │    │                 │
│  Gateway        │    │  Controller     │    │  Infrastructure │
│  (Port: 8080)   │───▶│  (Port: 8081)   │───▶│  MySQL/Redis/   │
│                 │    │                 │    │  MinIO          │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 服务说明

### 应用服务
- **online-judge-gateway**: 网关服务，端口 8080，负责请求路由和负载均衡
- **online-judge-controller**: 控制台服务，端口 8081，核心业务逻辑

### 基础设施服务
- **MySQL 8.0**: 数据库服务，端口 3306
- **Redis 7**: 缓存服务，端口 6379
- **MinIO**: 对象存储服务，端口 9000 (API) / 9001 (控制台)

## 快速开始

### 1. 环境准备

确保已安装 Docker 和 Docker Compose：
```bash
docker --version
docker-compose --version
```

### 2. 配置环境变量

编辑根目录下的 `.env` 文件，设置 MinIO 访问密钥：
```env
MINIO_ACCESS_KEY_ID=your_access_key
MINIO_SECRET_ACCESS_KEY=your_secret_key
```

### 3. 启动所有服务

在项目根目录执行：
```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

### 4. 验证服务

- **网关服务**: http://localhost:8080/health
- **控制台服务**: http://localhost:8081/health
- **MinIO 控制台**: http://localhost:9001
- **MySQL**: localhost:3306 (用户名: oj, 密码: 123456)
- **Redis**: localhost:6379

## 服务配置

### 网关服务配置
- 配置文件目录: `./gateway-config`
- 日志目录: `./gateway-log`
- 健康检查: `/health` 端点

### 控制台服务配置
- 配置文件目录: `./controller-config`
- 日志目录: `./controller-log`
- 环境变量文件: `./online_judge_controller/.env`
- 健康检查: `/health` 端点

### 数据库配置
- 数据库名: `online_judge`
- 用户名: `oj`
- 密码: `123456`
- Root 密码: `rootpassword`

### MinIO 配置
- 访问密钥: 从 `.env` 文件读取
- 数据目录: Docker volume `minio_data`
- 控制台地址: http://localhost:9001

## 常用命令

```bash
# 启动服务
docker-compose up -d

# 停止服务
docker-compose down

# 重启特定服务
docker-compose restart online-judge-gateway

# 查看服务日志
docker-compose logs -f online-judge-controller

# 进入服务容器
docker-compose exec online-judge-gateway sh

# 重新构建镜像
docker-compose build --no-cache

# 清理所有数据（谨慎使用）
docker-compose down -v
```

## 目录结构

```
OnlineJudge/
├── docker-compose.yaml          # Docker Compose 配置文件
├── .env                        # 环境变量文件
├── README-Docker.md            # 本文档
├── online_judge_gateway/       # 网关服务源码
├── online_judge_controller/    # 控制台服务源码
├── gateway-config/            # 网关配置目录（运行时创建）
├── gateway-log/               # 网关日志目录（运行时创建）
├── controller-config/         # 控制台配置目录（运行时创建）
├── controller-log/            # 控制台日志目录（运行时创建）
└── mysql-init/                # MySQL 初始化脚本目录（可选）
```

## 故障排除

### 1. 服务启动失败
```bash
# 查看详细日志
docker-compose logs service-name

# 检查服务状态
docker-compose ps
```

### 2. 端口冲突
如果端口被占用，可以修改 `docker-compose.yaml` 中的端口映射：
```yaml
ports:
  - "8080:8080"  # 改为 "8082:8080"
```

### 3. 数据持久化
所有数据都存储在 Docker volumes 中：
- `mysql_data`: MySQL 数据
- `redis_data`: Redis 数据
- `minio_data`: MinIO 数据

### 4. 网络连接问题
所有服务都在 `online-judge-network` 网络中，服务间可以通过服务名互相访问。

## 开发模式

如果需要在开发模式下运行：

1. 修改 `docker-compose.yaml` 中的 volumes 配置，挂载源码目录
2. 使用 `docker-compose up` (不加 -d) 查看实时日志
3. 修改代码后使用 `docker-compose restart service-name` 重启服务

## 生产部署建议

1. 修改默认密码和密钥
2. 使用外部数据库和缓存服务
3. 配置反向代理（如 Nginx）
4. 启用 HTTPS
5. 配置日志轮转
6. 设置资源限制
7. 配置监控和告警