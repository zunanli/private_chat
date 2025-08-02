# AI Chat Backend

基于NestJS框架的AI聊天后端服务，支持多种AI模型集成。

## 🛠️ 技术栈

- **框架**: NestJS + TypeScript
- **数据库**: MySQL 8.0 + TypeORM
- **缓存**: Redis 7
- **AI集成**: OpenAI、Google GenAI、LangChain
- **容器化**: Docker + Docker Compose

## 🚀 快速启动

### 方式一：混合开发模式 (推荐)

适合日常开发，数据库用Docker，应用在本地运行：

```bash
# 1. 启动基础服务 (MySQL + Redis)
docker-compose up mysql redis -d

# 2. 配置环境变量
cp .env.example src/.env
# 编辑 src/.env 文件，配置必要的环境变量

# 3. 安装依赖并启动应用
pnpm install
pnpm run start:dev
```

### 方式二：完全容器化开发

所有服务都在Docker中运行：

```bash
# 1. 配置环境变量
cp .env.example src/.env

# 2. 使用开发版docker-compose启动
docker-compose -f docker-compose.dev.yml up --build

# 或者后台运行
docker-compose -f docker-compose.dev.yml up --build -d
```

### 方式三：生产环境部署

```bash
# 构建并启动生产环境
docker-compose up --build -d
```

## 📝 环境配置

### 必需的环境变量

复制 `.env.example` 为 `src/.env` 并配置：

```bash
# 数据库配置
DB_HOST=localhost          # Docker模式下使用 mysql
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=630wujiayuwy
DB_DATABASE=aiChat

# Redis配置  
redis_server_host=localhost  # Docker模式下使用 redis
redis_server_port=6379
redis_server_db=0

# AI服务密钥（需要自己申请）
OPENAI_API_KEY=your_openai_api_key_here
GOOGLE_API_KEY=your_google_api_key_here
```

## 🐳 Docker 使用说明

### 常用命令

```bash
# 查看运行中的容器
docker-compose ps

# 查看日志
docker-compose logs app
docker-compose logs mysql

# 停止所有服务
docker-compose down

# 停止并删除数据卷（谨慎使用）
docker-compose down -v

# 重新构建应用镜像
docker-compose build app

# 进入容器内部
docker exec -it aichat-app-dev bash
docker exec -it aichat-mysql-dev mysql -uroot -p
```

### 数据持久化

- MySQL数据: `mysql_data` 卷
- Redis数据: `redis_data` 卷
- 应用上传文件: `./uploads` 目录

## 🔧 开发工具

```bash
# 代码格式化
pnpm run format

# 代码检查
pnpm run lint

# 运行测试
pnpm run test

# 构建生产版本
pnpm run build
```

## 📡 API接口

应用启动后访问：
- 主接口: http://localhost:3000
- 静态文件: http://localhost:3000/uploads/

## 🗂️ 项目结构

```
src/
├── agent/          # AI代理模块
├── ai/             # AI服务集成
├── chat/           # 聊天功能
├── email/          # 邮件服务
├── file/           # 文件处理
├── redis/          # Redis缓存
├── users/          # 用户管理
├── filters/        # 全局过滤器
├── interceptors/   # 拦截器
└── main.ts         # 应用入口
```

## 📋 开发注意事项

1. **首次启动**：确保Docker已安装并启动
2. **端口占用**：确保3306、6379、3000端口未被占用
3. **AI密钥**：需要自己申请OpenAI和Google的API密钥
4. **数据库**：首次启动会自动创建数据库和表结构
