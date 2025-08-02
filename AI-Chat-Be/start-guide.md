# AI Chat Backend 启动指南

## 🚨 当前架构的问题

你发现的问题完全正确：

1. **现有 docker-compose.yml 不完整**：只包含 MySQL 和 Redis
2. **Dockerfile 是孤立的**：没有被 docker-compose 使用
3. **混合部署模式**：基础设施容器化，应用本地运行

## 📋 当前的启动方式（原设计）

### 1. 启动基础设施服务
```bash
# 只启动数据库和缓存
docker-compose up mysql redis -d
```

### 2. 本地启动应用
```bash
# 安装依赖
pnpm install

# 配置环境变量（需要创建 src/.env 文件）
# DB_HOST=localhost
# DB_PORT=3306
# DB_USERNAME=root
# DB_PASSWORD=630wujiayuwy
# DB_DATABASE=aiChat

# 启动应用
pnpm run start:dev
```

## 🛠️ 完整容器化解决方案

### 方案一：使用完整的 docker-compose

```bash
# 使用新的完整配置
docker-compose -f docker-compose.complete.yml up -d

# 或者修改现有文件后
docker-compose up -d
```

### 方案二：分步骤容器化

```bash
# 1. 构建应用镜像
docker build -t ai-chat-be:latest .

# 2. 启动基础设施
docker-compose up mysql redis -d

# 3. 运行应用容器
docker run -d \
  --name ai-chat-app \
  --network aichat-network \
  -p 3000:3000 \
  -e DB_HOST=mysql \
  -e DB_PORT=3306 \
  -e DB_USERNAME=root \
  -e DB_PASSWORD=630wujiayuwy \
  -e DB_DATABASE=aiChat \
  ai-chat-be:latest
```

## 🔧 需要解决的配置问题

### 1. 创建环境变量文件
```bash
# 创建 src/.env
cat > src/.env << EOF
DB_HOST=localhost
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=630wujiayuwy
DB_DATABASE=aiChat
EOF
```

### 2. 修改数据库连接配置
在容器环境中，需要将 `DB_HOST` 改为 `mysql`（服务名）

### 3. 创建uploads目录
```bash
mkdir -p uploads
```

## 🎯 推荐的项目结构改进

```
AI-Chat-Be/
├── docker-compose.yml          # 完整的服务编排
├── docker-compose.dev.yml      # 开发环境配置
├── docker-compose.prod.yml     # 生产环境配置
├── Dockerfile                  # 开发用镜像
├── Dockerfile.production       # 生产优化镜像
├── .env.example               # 环境变量模板
├── .env.local                 # 本地开发配置
└── scripts/
    ├── start-dev.sh           # 开发启动脚本
    └── start-prod.sh          # 生产启动脚本
```

## 🚀 快速启动（推荐）

### 开发环境
```bash
# 1. 复制环境变量
cp .env.example .env.local

# 2. 启动所有服务
docker-compose -f docker-compose.complete.yml up -d

# 3. 查看日志
docker-compose -f docker-compose.complete.yml logs -f app
```

### 本地开发调试
```bash
# 1. 只启动基础设施
docker-compose up mysql redis -d

# 2. 本地启动应用（支持热重载）
pnpm run start:dev
```

这样就能解决你提到的所有问题！