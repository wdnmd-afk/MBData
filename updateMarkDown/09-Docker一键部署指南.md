# Metabase Docker 一键部署指南

> 使用 Docker 快速部署 Metabase 项目,5 分钟完成!

---

## 目录

- [快速开始](#快速开始)
- [部署方式对比](#部署方式对比)
- [方式1: 快速部署](#方式1-快速部署推荐)
- [方式2: 完整构建](#方式2-完整构建)
- [方式3: 开发模式](#方式3-开发模式)
- [配置说明](#配置说明)
- [常见问题](#常见问题)
- [故障排查](#故障排查)

---

## 快速开始

### 前置要求

| 软件 | 版本要求 | 安装链接 |
|------|---------|---------|
| **Docker** | >= 20.10 | [安装 Docker](https://docs.docker.com/get-docker/) |
| **Docker Compose** | >= 2.0 | [安装 Compose](https://docs.docker.com/compose/install/) |

### 验证安装

```bash
# 检查 Docker 版本
docker --version
# 输出: Docker version 24.0.0 或更高

# 检查 Docker Compose 版本
docker-compose --version
# 输出: Docker Compose version v2.20.0 或更高

# 检查 Docker 是否运行
docker ps
# 应该显示容器列表(可能为空)
```

---

## 部署方式对比

| 方式 | 耗时 | 镜像大小 | 适用场景 | 推荐度 |
|------|------|---------|---------|--------|
| **快速部署** | 5 分钟 | ~500MB | 生产环境、快速体验 | ⭐⭐⭐⭐⭐ |
| **完整构建** | 30-60 分钟 | ~1.5GB | 自定义修改、完全控制 | ⭐⭐⭐ |
| **开发模式** | 10 分钟 | ~800MB | 本地开发、调试 | ⭐⭐⭐⭐ |

---

## 方式1: 快速部署(推荐)

### 特点

- ✅ 使用官方预构建镜像
- ✅ 5 分钟完成部署
- ✅ 适合生产环境
- ✅ 镜像体积小

### 一键部署

```bash
# 方法 A: 使用脚本(推荐)
chmod +x docker-deploy.sh
./docker-deploy.sh
# 选择选项 1

# 方法 B: 手动执行
docker-compose -f docker-compose-quick.yml up -d
```

### 详细步骤

#### 步骤 1: 创建配置文件

创建 `docker-compose-quick.yml`:

```yaml
version: '3.9'

services:
  # Metabase 应用
  metabase:
    image: metabase/metabase:latest
    container_name: metabase-app
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      # 数据库配置
      MB_DB_TYPE: postgres
      MB_DB_DBNAME: metabase
      MB_DB_PORT: 5432
      MB_DB_USER: metabase
      MB_DB_PASS: metabase_password
      MB_DB_HOST: postgres
      
      # 时区设置
      JAVA_TIMEZONE: Asia/Shanghai
      TZ: Asia/Shanghai
    volumes:
      - ./plugins:/plugins
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - metabase-network

  # PostgreSQL 数据库
  postgres:
    image: postgres:15-alpine
    container_name: metabase-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: metabase
      POSTGRES_PASSWORD: metabase_password
      POSTGRES_DB: metabase
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - metabase-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U metabase"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres-data:

networks:
  metabase-network:
```

#### 步骤 2: 启动服务

```bash
docker-compose -f docker-compose-quick.yml up -d
```

**输出**:
```
Creating network "metabase_metabase-network" ... done
Creating volume "metabase_postgres-data" ... done
Creating metabase-postgres ... done
Creating metabase-app      ... done
```

#### 步骤 3: 查看状态

```bash
docker-compose -f docker-compose-quick.yml ps
```

**预期输出**:
```
NAME                 STATUS              PORTS
metabase-app         Up 2 minutes        0.0.0.0:3000->3000/tcp
metabase-postgres    Up 2 minutes        5432/tcp
```

#### 步骤 4: 访问 Metabase

打开浏览器访问: **http://localhost:3000**

**首次访问会看到设置向导**:
1. 选择语言: 中文
2. 创建管理员账户
3. 添加数据源(可选)
4. 完成设置

---

## 方式2: 完整构建

### 特点

- ✅ 从源代码构建
- ✅ 完全控制构建过程
- ✅ 可以自定义修改
- ⚠️ 耗时较长(30-60 分钟)

### 一键构建

```bash
# 方法 A: 使用脚本
chmod +x docker-deploy.sh
./docker-deploy.sh
# 选择选项 2

# 方法 B: 手动执行
docker-compose -f docker-compose-simple.yml build
docker-compose -f docker-compose-simple.yml up -d
```

### 详细步骤

#### 步骤 1: 构建镜像

```bash
# 构建 Metabase 镜像
docker-compose -f docker-compose-simple.yml build --no-cache

# 构建过程会执行:
# 1. 安装 Node.js 依赖
# 2. 安装 Java 和 Clojure
# 3. 编译前端代码
# 4. 编译后端代码
# 5. 打包成 JAR 文件
# 6. 创建运行时镜像
```

**预计耗时**: 30-60 分钟(取决于网络和硬件)

#### 步骤 2: 启动服务

```bash
docker-compose -f docker-compose-simple.yml up -d
```

#### 步骤 3: 查看构建日志

```bash
# 查看构建进度
docker-compose -f docker-compose-simple.yml logs -f metabase
```

---

## 方式3: 开发模式

### 特点

- ✅ 适合本地开发
- ✅ 只启动数据库容器
- ✅ Metabase 在本地运行
- ✅ 支持热更新

### 一键启动

```bash
# 方法 A: 使用脚本
chmod +x docker-deploy.sh
./docker-deploy.sh
# 选择选项 3

# 方法 B: 手动执行
COMPOSE_PROFILES=postgresappdb docker-compose --file dev/docker-compose.yml up -d
```

### 详细步骤

#### 步骤 1: 启动数据库

```bash
# 启动 PostgreSQL 数据库
COMPOSE_PROFILES=postgresappdb docker-compose --file dev/docker-compose.yml up -d
```

#### 步骤 2: 本地启动 Metabase

**终端 1 - 启动后端**:
```bash
clojure -M:run
```

**终端 2 - 启动前端**:
```bash
yarn build-hot
```

#### 步骤 3: 访问应用

- **Metabase**: http://localhost:3000
- **PostgreSQL**: localhost:5432
  - 用户名: `mbuser`
  - 密码: `password`
  - 数据库: `metabase`

---

## 配置说明

### 环境变量

#### 数据库配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `MB_DB_TYPE` | 数据库类型 | `postgres` |
| `MB_DB_HOST` | 数据库主机 | `postgres` |
| `MB_DB_PORT` | 数据库端口 | `5432` |
| `MB_DB_DBNAME` | 数据库名称 | `metabase` |
| `MB_DB_USER` | 数据库用户 | `metabase` |
| `MB_DB_PASS` | 数据库密码 | `metabase_password` |

#### 应用配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `MB_JETTY_HOST` | 监听地址 | `0.0.0.0` |
| `MB_JETTY_PORT` | 监听端口 | `3000` |
| `JAVA_TIMEZONE` | Java 时区 | `Asia/Shanghai` |
| `TZ` | 系统时区 | `Asia/Shanghai` |

#### 邮件配置(可选)

```yaml
environment:
  MB_EMAIL_SMTP_HOST: smtp.gmail.com
  MB_EMAIL_SMTP_PORT: 587
  MB_EMAIL_SMTP_USERNAME: your-email@gmail.com
  MB_EMAIL_SMTP_PASSWORD: your-app-password
  MB_EMAIL_FROM_ADDRESS: metabase@yourdomain.com
  MB_EMAIL_SMTP_SECURITY: tls
```

### 端口映射

| 容器端口 | 主机端口 | 说明 |
|---------|---------|------|
| 3000 | 3000 | Metabase Web 界面 |
| 5432 | 5432 | PostgreSQL(仅开发模式) |

### 数据持久化

```yaml
volumes:
  # PostgreSQL 数据
  postgres-data:
    driver: local
  
  # Metabase 插件(可选)
  ./plugins:/plugins
```

**数据存储位置**:
- Linux/Mac: `/var/lib/docker/volumes/`
- Windows: `C:\ProgramData\Docker\volumes\`

---

## 常用命令

### 启动和停止

```bash
# 启动服务(后台运行)
docker-compose -f docker-compose-quick.yml up -d

# 启动服务(前台运行,查看日志)
docker-compose -f docker-compose-quick.yml up

# 停止服务
docker-compose -f docker-compose-quick.yml stop

# 停止并删除容器
docker-compose -f docker-compose-quick.yml down

# 停止并删除容器和数据卷
docker-compose -f docker-compose-quick.yml down -v
```

### 查看日志

```bash
# 查看所有服务日志
docker-compose -f docker-compose-quick.yml logs

# 查看 Metabase 日志
docker-compose -f docker-compose-quick.yml logs metabase

# 实时跟踪日志
docker-compose -f docker-compose-quick.yml logs -f metabase

# 查看最后 100 行日志
docker-compose -f docker-compose-quick.yml logs --tail=100 metabase
```

### 查看状态

```bash
# 查看容器状态
docker-compose -f docker-compose-quick.yml ps

# 查看容器详细信息
docker inspect metabase-app

# 查看资源使用情况
docker stats metabase-app metabase-postgres
```

### 进入容器

```bash
# 进入 Metabase 容器
docker exec -it metabase-app /bin/bash

# 进入 PostgreSQL 容器
docker exec -it metabase-postgres /bin/bash

# 连接 PostgreSQL 数据库
docker exec -it metabase-postgres psql -U metabase -d metabase
```

### 备份和恢复

#### 备份 PostgreSQL 数据

```bash
# 备份数据库
docker exec metabase-postgres pg_dump -U metabase metabase > metabase_backup_$(date +%Y%m%d).sql

# 或使用 docker-compose
docker-compose -f docker-compose-quick.yml exec postgres pg_dump -U metabase metabase > backup.sql
```

#### 恢复 PostgreSQL 数据

```bash
# 恢复数据库
docker exec -i metabase-postgres psql -U metabase metabase < metabase_backup_20251120.sql

# 或使用 docker-compose
docker-compose -f docker-compose-quick.yml exec -T postgres psql -U metabase metabase < backup.sql
```

---

## 常见问题

### Q1: 容器启动失败,提示端口被占用

**错误信息**:
```
Error starting userland proxy: listen tcp4 0.0.0.0:3000: bind: address already in use
```

**解决方案**:

```bash
# 方法 1: 查找占用端口的进程
# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# Linux/Mac
lsof -i :3000
kill -9 <PID>

# 方法 2: 修改端口映射
# 编辑 docker-compose-quick.yml
ports:
  - "3001:3000"  # 改为 3001 端口
```

### Q2: Metabase 启动很慢

**原因**: 首次启动需要初始化数据库

**解决方案**:

```bash
# 查看启动日志
docker-compose -f docker-compose-quick.yml logs -f metabase

# 等待看到以下信息:
# Metabase Initialization COMPLETE
```

**预计耗时**: 2-5 分钟

### Q3: 忘记管理员密码

**解决方案**:

```bash
# 1. 进入 PostgreSQL 容器
docker exec -it metabase-postgres psql -U metabase metabase

# 2. 重置密码(将密码重置为 'password')
UPDATE core_user 
SET password = '$2a$10$VBCJKZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZe' 
WHERE email = 'admin@example.com';

# 3. 退出
\q
```

### Q4: 如何升级 Metabase

**快速部署模式**:

```bash
# 1. 停止服务
docker-compose -f docker-compose-quick.yml down

# 2. 拉取最新镜像
docker pull metabase/metabase:latest

# 3. 启动服务
docker-compose -f docker-compose-quick.yml up -d
```

**完整构建模式**:

```bash
# 1. 拉取最新代码
git pull origin main

# 2. 重新构建
docker-compose -f docker-compose-simple.yml build --no-cache

# 3. 启动服务
docker-compose -f docker-compose-simple.yml up -d
```

### Q5: 如何更换数据库为 MySQL

**修改 docker-compose-quick.yml**:

```yaml
services:
  metabase:
    environment:
      MB_DB_TYPE: mysql
      MB_DB_DBNAME: metabase
      MB_DB_PORT: 3306
      MB_DB_USER: metabase
      MB_DB_PASS: metabase_password
      MB_DB_HOST: mysql
    depends_on:
      mysql:
        condition: service_healthy

  mysql:
    image: mysql:8.0
    container_name: metabase-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: metabase
      MYSQL_USER: metabase
      MYSQL_PASSWORD: metabase_password
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - metabase-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mysql-data:
```

---

## 故障排查

### 检查容器状态

```bash
# 查看所有容器
docker ps -a

# 查看容器日志
docker logs metabase-app

# 查看容器详细信息
docker inspect metabase-app
```

### 检查网络连接

```bash
# 查看网络
docker network ls

# 查看网络详情
docker network inspect metabase_metabase-network

# 测试容器间连接
docker exec metabase-app ping postgres
```

### 检查数据卷

```bash
# 查看数据卷
docker volume ls

# 查看数据卷详情
docker volume inspect metabase_postgres-data

# 清理未使用的数据卷
docker volume prune
```

### 完全重置

```bash
# 1. 停止并删除所有容器
docker-compose -f docker-compose-quick.yml down -v

# 2. 删除镜像
docker rmi metabase/metabase:latest
docker rmi postgres:15-alpine

# 3. 清理系统
docker system prune -a

# 4. 重新部署
docker-compose -f docker-compose-quick.yml up -d
```

---

## 生产环境建议

### 1. 使用环境变量文件

创建 `.env` 文件:

```bash
# 数据库配置
POSTGRES_USER=metabase
POSTGRES_PASSWORD=your_secure_password_here
POSTGRES_DB=metabase

# Metabase 配置
MB_DB_PASS=your_secure_password_here
JAVA_TIMEZONE=Asia/Shanghai
```

修改 `docker-compose-quick.yml`:

```yaml
services:
  metabase:
    environment:
      MB_DB_PASS: ${MB_DB_PASS}
  
  postgres:
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

### 2. 使用 Docker Secrets

```yaml
services:
  metabase:
    secrets:
      - db_password
    environment:
      MB_DB_PASS_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### 3. 配置资源限制

```yaml
services:
  metabase:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '1.0'
          memory: 1G
```

### 4. 配置健康检查

```yaml
services:
  metabase:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 60s
```

### 5. 使用反向代理

**Nginx 配置示例**:

```nginx
server {
    listen 80;
    server_name metabase.yourdomain.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 6. 配置 HTTPS

使用 Let's Encrypt:

```bash
# 安装 Certbot
sudo apt install certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d metabase.yourdomain.com
```

---

## 性能优化

### 1. 增加 Java 堆内存

```yaml
services:
  metabase:
    environment:
      JAVA_OPTS: "-Xmx2g -Xms1g"
```

### 2. 使用 SSD 存储

```yaml
volumes:
  postgres-data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /path/to/ssd/metabase-data
```

### 3. 配置 PostgreSQL 性能参数

```yaml
services:
  postgres:
    command:
      - postgres
      - -c
      - shared_buffers=256MB
      - -c
      - max_connections=200
      - -c
      - effective_cache_size=1GB
```

---

## 总结

### 快速参考

**快速部署(推荐)**:
```bash
chmod +x docker-deploy.sh && ./docker-deploy.sh
# 选择选项 1
```

**访问应用**:
```
http://localhost:3000
```

**查看日志**:
```bash
docker-compose -f docker-compose-quick.yml logs -f metabase
```

**停止服务**:
```bash
docker-compose -f docker-compose-quick.yml down
```

### 优势

- ✅ 5 分钟完成部署
- ✅ 自动配置数据库
- ✅ 数据持久化
- ✅ 易于维护和升级
- ✅ 适合生产环境

---

**现在就开始部署吧!** 🚀
