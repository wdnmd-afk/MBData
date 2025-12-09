# Docker 网络问题解决指南

> 解决镜像拉取超时、网络连接失败等问题

---

## 目录

- [问题诊断](#问题诊断)
- [方案1: 配置镜像加速器](#方案1-配置镜像加速器推荐)
- [方案2: 使用代理](#方案2-使用代理)
- [方案3: 手动下载镜像](#方案3-手动下载镜像)
- [方案4: 使用本地镜像](#方案4-使用本地镜像)
- [验证和测试](#验证和测试)

---

## 问题诊断

### 你遇到的错误

```
failed to copy: httpReadSeeker: failed open: failed to do request: 
Get "https://docker-images-prod...": net/http: TLS handshake timeout
```

### 错误分析

| 问题 | 原因 | 影响 |
|------|------|------|
| **TLS handshake timeout** | 网络连接超时 | 无法下载镜像 |
| **Docker Hub 访问慢** | 国内网络限制 | 下载速度慢 |
| **DNS 解析失败** | DNS 配置问题 | 无法连接服务器 |

---

## 方案1: 配置镜像加速器(推荐)

### 为什么需要镜像加速器?

- ✅ 加速镜像下载速度
- ✅ 提高成功率
- ✅ 节省时间
- ✅ 稳定可靠

### 国内镜像源对比

| 镜像源 | 地址 | 速度 | 稳定性 | 推荐度 |
|--------|------|------|--------|--------|
| **阿里云** | 需注册获取 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **腾讯云** | mirror.ccs.tencentyun.com | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **网易云** | hub-mirror.c.163.com | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **中科大** | docker.mirrors.ustc.edu.cn | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

### Windows 配置步骤

#### 方法 A: 使用自动化脚本(推荐)

```powershell
# 运行配置脚本
.\setup-docker-mirror.ps1

# 选择镜像源
# 推荐选择: 6 - 配置多个镜像源
```

#### 方法 B: 手动配置

**步骤 1: 打开 Docker Desktop**

1. 启动 Docker Desktop
2. 点击右上角 ⚙️ 设置图标

**步骤 2: 进入 Docker Engine 配置**

1. 选择左侧菜单 "Docker Engine"
2. 看到 JSON 编辑器

**步骤 3: 添加镜像加速器配置**

找到或添加 `registry-mirrors` 配置:

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://registry.docker-cn.com"
  ]
}
```

**步骤 4: 应用配置**

1. 点击右下角 "Apply & Restart"
2. 等待 Docker Desktop 重启(30-60秒)

---

### 阿里云镜像加速器(最快)

#### 获取专属加速地址

1. **访问**: https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
2. **登录** 阿里云账号(没有账号需要注册)
3. **复制** 你的专属加速地址

**示例地址**:
```
https://xxxxx.mirror.aliyuncs.com
```

#### 配置阿里云加速器

```json
{
  "registry-mirrors": [
    "https://xxxxx.mirror.aliyuncs.com"
  ]
}
```

**优势**:
- ✅ 速度最快
- ✅ 稳定性最好
- ✅ 专属加速地址
- ✅ 免费使用

---

### Linux 配置步骤

#### 方法 A: 使用脚本

```bash
chmod +x setup-docker-mirror.sh
./setup-docker-mirror.sh
```

#### 方法 B: 手动配置

```bash
# 1. 创建配置目录
sudo mkdir -p /etc/docker

# 2. 编辑配置文件
sudo nano /etc/docker/daemon.json

# 3. 添加以下内容
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://registry.docker-cn.com"
  ]
}

# 4. 重启 Docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# 5. 验证配置
docker info | grep -A 10 "Registry Mirrors"
```

---

### macOS 配置步骤

**步骤 1-4**: 与 Windows 相同

1. 打开 Docker Desktop
2. 点击设置图标
3. 选择 "Docker Engine"
4. 添加镜像加速器配置
5. 点击 "Apply & Restart"

---

## 方案2: 使用代理

### 配置 Docker 代理

#### Windows (Docker Desktop)

**步骤 1: 打开设置**

1. Docker Desktop -> 设置
2. Resources -> Proxies

**步骤 2: 配置代理**

```
HTTP Proxy: http://127.0.0.1:7890
HTTPS Proxy: http://127.0.0.1:7890
```

**步骤 3: 应用配置**

点击 "Apply & Restart"

---

#### Linux

```bash
# 1. 创建代理配置目录
sudo mkdir -p /etc/systemd/system/docker.service.d

# 2. 创建代理配置文件
sudo nano /etc/systemd/system/docker.service.d/http-proxy.conf

# 3. 添加以下内容
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,127.0.0.1"

# 4. 重启 Docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# 5. 验证配置
sudo systemctl show --property=Environment docker
```

---

## 方案3: 手动下载镜像

### 使用国内镜像仓库

#### 步骤 1: 从国内源拉取镜像

```bash
# 使用阿里云镜像
docker pull registry.cn-hangzhou.aliyuncs.com/library/postgres:15-alpine

# 重新标记为原始名称
docker tag registry.cn-hangzhou.aliyuncs.com/library/postgres:15-alpine postgres:15-alpine

# 删除临时标签
docker rmi registry.cn-hangzhou.aliyuncs.com/library/postgres:15-alpine
```

#### 步骤 2: 拉取 Metabase 镜像

```bash
# 使用阿里云镜像
docker pull registry.cn-hangzhou.aliyuncs.com/library/metabase:latest

# 重新标记
docker tag registry.cn-hangzhou.aliyuncs.com/library/metabase:latest metabase/metabase:latest

# 删除临时标签
docker rmi registry.cn-hangzhou.aliyuncs.com/library/metabase:latest
```

---

### 创建一键下载脚本

创建 `pull-images.sh`:

```bash
#!/bin/bash

echo "========================================="
echo "下载 Metabase 所需镜像"
echo "========================================="
echo ""

# 使用中科大镜像源
MIRROR="docker.mirrors.ustc.edu.cn"

echo "1. 下载 PostgreSQL 镜像..."
docker pull postgres:15-alpine

if [ $? -ne 0 ]; then
    echo "❌ 下载失败,尝试使用镜像加速器..."
    echo "请先配置镜像加速器: ./setup-docker-mirror.sh"
    exit 1
fi

echo "✅ PostgreSQL 镜像下载完成"
echo ""

echo "2. 下载 Metabase 镜像..."
docker pull metabase/metabase:latest

if [ $? -ne 0 ]; then
    echo "❌ 下载失败"
    exit 1
fi

echo "✅ Metabase 镜像下载完成"
echo ""

echo "========================================="
echo "✅ 所有镜像下载完成!"
echo "========================================="
echo ""
echo "查看已下载的镜像:"
docker images | grep -E "postgres|metabase"
echo ""
echo "现在可以运行部署脚本:"
echo "./docker-deploy.sh"
echo ""
```

使用方法:

```bash
chmod +x pull-images.sh
./pull-images.sh
```

---

## 方案4: 使用本地镜像

### 从其他机器导出镜像

#### 在有网络的机器上

```bash
# 1. 拉取镜像
docker pull postgres:15-alpine
docker pull metabase/metabase:latest

# 2. 导出镜像
docker save postgres:15-alpine -o postgres-15-alpine.tar
docker save metabase/metabase:latest -o metabase-latest.tar

# 3. 压缩镜像文件(可选)
gzip postgres-15-alpine.tar
gzip metabase-latest.tar
```

#### 在目标机器上

```bash
# 1. 复制 tar 文件到目标机器

# 2. 解压(如果压缩了)
gunzip postgres-15-alpine.tar.gz
gunzip metabase-latest.tar.gz

# 3. 导入镜像
docker load -i postgres-15-alpine.tar
docker load -i metabase-latest.tar

# 4. 验证
docker images | grep -E "postgres|metabase"

# 5. 运行部署
./docker-deploy.sh
```

---

## 验证和测试

### 验证镜像加速器配置

```bash
# 查看 Docker 配置
docker info

# 查找 Registry Mirrors 部分
docker info | grep -A 10 "Registry Mirrors"
```

**正确输出**:
```
Registry Mirrors:
  https://docker.mirrors.ustc.edu.cn/
  https://hub-mirror.c.163.com/
  https://registry.docker-cn.com/
```

---

### 测试镜像下载速度

```bash
# 删除测试镜像(如果存在)
docker rmi alpine:latest 2>/dev/null

# 测试下载
time docker pull alpine:latest
```

**预期结果**:
- 配置前: 60-300 秒或超时
- 配置后: 5-30 秒

---

### 测试 Metabase 部署

```bash
# 1. 清理之前的失败尝试
docker-compose -f docker-compose-quick.yml down -v

# 2. 重新部署
./docker-deploy.sh
# 选择选项 1

# 3. 查看日志
docker-compose -f docker-compose-quick.yml logs -f
```

---

## 常见问题

### Q1: 配置镜像加速器后仍然很慢

**原因**: 镜像源可能不稳定

**解决方案**:

```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://registry.docker-cn.com",
    "https://mirror.ccs.tencentyun.com"
  ]
}
```

配置多个镜像源,Docker 会自动选择最快的。

---

### Q2: 提示 "unauthorized: authentication required"

**原因**: 镜像源需要认证

**解决方案**:

```bash
# 登录 Docker Hub
docker login

# 或使用阿里云镜像(不需要认证)
```

---

### Q3: 镜像下载到一半失败

**原因**: 网络不稳定

**解决方案**:

```bash
# 1. 清理失败的下载
docker system prune -a

# 2. 重新下载
docker pull postgres:15-alpine

# 3. 如果仍然失败,尝试分段下载
# 使用更小的基础镜像
docker pull postgres:15-alpine
```

---

### Q4: DNS 解析失败

**错误**:
```
Could not resolve host: docker.io
```

**解决方案**:

**Windows**:

1. Docker Desktop -> 设置
2. Docker Engine
3. 添加 DNS 配置:

```json
{
  "dns": ["8.8.8.8", "114.114.114.114"]
}
```

**Linux**:

```bash
# 编辑 Docker 配置
sudo nano /etc/docker/daemon.json

# 添加 DNS
{
  "dns": ["8.8.8.8", "114.114.114.114"]
}

# 重启 Docker
sudo systemctl restart docker
```

---

## 性能优化

### 并发下载

```json
{
  "max-concurrent-downloads": 10,
  "max-concurrent-uploads": 10
}
```

### 存储驱动优化

```json
{
  "storage-driver": "overlay2"
}
```

### 完整优化配置

```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com"
  ],
  "dns": ["8.8.8.8", "114.114.114.114"],
  "max-concurrent-downloads": 10,
  "max-concurrent-uploads": 10,
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

---

## 故障排查流程

### 步骤 1: 检查网络连接

```bash
# 测试网络
ping docker.io
ping registry-1.docker.io

# 测试 HTTPS
curl -I https://registry-1.docker.io/v2/
```

### 步骤 2: 检查 Docker 配置

```bash
# 查看配置
docker info

# 查看镜像加速器
docker info | grep "Registry Mirrors"

# 查看 DNS
docker info | grep "DNS"
```

### 步骤 3: 测试镜像下载

```bash
# 测试小镜像
docker pull alpine:latest

# 如果成功,测试大镜像
docker pull postgres:15-alpine
```

### 步骤 4: 查看详细日志

**Windows**:
```
C:\Users\<用户名>\AppData\Local\Docker\log.txt
```

**Linux**:
```bash
sudo journalctl -u docker -f
```

---

## 总结

### 推荐方案

**最佳方案**: 配置多个国内镜像加速器

```powershell
# Windows
.\setup-docker-mirror.ps1
# 选择选项 6 - 配置多个镜像源
```

### 配置后效果

| 项目 | 配置前 | 配置后 |
|------|--------|--------|
| **下载速度** | 10-50 KB/s | 1-10 MB/s |
| **成功率** | 30-50% | 95-99% |
| **下载时间** | 5-30 分钟 | 30-120 秒 |

### 立即修复

```powershell
# 1. 配置镜像加速器
.\setup-docker-mirror.ps1

# 2. 重启 Docker Desktop

# 3. 验证配置
docker info | grep "Registry Mirrors"

# 4. 重新部署
./docker-deploy.sh
```

---

**现在重新尝试部署,应该会快很多!** 🚀
