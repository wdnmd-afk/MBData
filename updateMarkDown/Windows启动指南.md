# Metabase Windows 启动指南

> 更新时间: 2025-11-19  
> 适用系统: Windows 10/11  
> 适用环境: Git Bash / PowerShell

---

## 📋 目录

- [环境准备](#环境准备)
- [快速启动](#快速启动)
- [常见问题](#常见问题)
- [故障排查](#故障排查)

---

## 🔧 环境准备

### 必需软件

| 软件 | 版本要求 | 安装方式 | 验证命令 |
|------|---------|---------|---------|
| Node.js | >= 22 | [官网下载](https://nodejs.org/) | `node --version` |
| Yarn | ^1.22.22 | `npm install -g yarn` | `yarn --version` |
| Java | >= 22 | [官网下载](https://adoptium.net/) | `java --version` |
| Clojure | 最新版 | [官网安装](https://clojure.org/guides/install_clojure) | `clojure --version` |

### 推荐工具

- **nrm**: npm 镜像管理 (`npm install -g nrm`)
- **Git Bash**: 命令行工具 (Git for Windows 自带)
- **VS Code**: 代码编辑器

---

## 🚀 快速启动

### 第一次启动 (完整流程)

#### 1. 克隆项目

```bash
git clone https://github.com/metabase/metabase.git
cd metabase
```

#### 2. 配置 npm 镜像 (可选,加速下载)

```bash
# 使用淘宝镜像
nrm use taobao

# 或手动设置
yarn config set registry https://registry.npmmirror.com
```

#### 3. 安装依赖

```bash
# 安装 Node 依赖
yarn install --ignore-scripts

# 安装 cross-env (Windows 兼容性)
yarn add -D cross-env
```

#### 4. 启动前端

**打开第一个 Git Bash 窗口**:

```bash
cd /e/github/metabase

# 启动前端开发服务器
yarn build-hot:js
```

**等待看到**:
```
✅ Rspack compiled successfully in XX.XX s
🌐 Project is running at: http://localhost:8080/
```

#### 5. 启动后端

**打开第二个 Git Bash 窗口**:

```bash
cd /e/github/metabase

# 启动后端服务器
clojure -M:run
```

**等待看到** (需要 3-5 分钟):
```
Starting Metabase...
Metabase Initialization COMPLETE
```

#### 6. 访问应用

打开浏览器访问: **http://localhost:3000**

---

### 日常启动 (已安装依赖)

只需要启动前后端服务:

```bash
# 窗口 1: 前端
yarn build-hot:js

# 窗口 2: 后端
clojure -M:run
```

---

## ❌ 常见问题

### 问题 1: `NODE_OPTIONS` 不是内部或外部命令

**错误信息**:
```
'NODE_OPTIONS' 不是内部或外部命令,也不是可运行的程序
```

**原因**: Git Bash 无法识别 Windows 环境变量设置语法

**解决方案**: 安装 `cross-env`

```bash
yarn add -D cross-env
```

然后修改 `package.json`:
```json
{
  "scripts": {
    "build-hot:js": "yarn && yarn clean-dev:js && cross-env NODE_OPTIONS=--max-old-space-size=8192 WEBPACK_BUNDLE=hot rspack serve"
  }
}
```

---

### 问题 2: `Cannot find module 'cljs/metabase.lib.js'`

**错误信息**:
```
ERROR in ../metabase-lib/aggregation.ts 1:0-43
× Cannot find module 'cljs/metabase.lib.js'
```

**原因**: 前端依赖 ClojureScript 编译生成的文件,但这些文件还未生成

**解决方案**: 先编译 ClojureScript

```bash
# 编译 ClojureScript (一次性,需要 5-10 分钟)
yarn build:cljs

# 然后启动前端
yarn build-hot:js
```

**或者**: 如果需要修改 ClojureScript 代码,使用热重载:

```bash
# 窗口 1: ClojureScript 热重载
yarn build-hot:cljs

# 窗口 2: 前端
yarn build-hot:js

# 窗口 3: 后端
clojure -M:run
```

---

### 问题 3: `yarn dev` 命令失败

**错误信息**:
```
['backend] ''clojure' 不是内部或外部命令
[frontend] 文件名、目录名或卷标语法不正确
```

**原因**: `concurrently` 在 Windows Git Bash 中路径解析有问题

**解决方案**: 不使用 `yarn dev`,改为分别启动各个服务 (见上方快速启动)

---

### 问题 4: 端口被占用

**错误信息**:
```
Error: listen EADDRINUSE: address already in use :::3000
```

**解决方案**: 查找并结束占用端口的进程

```bash
# 查找占用 3000 端口的进程
netstat -ano | grep 3000

# 结束进程 (在 PowerShell 中)
taskkill /F /PID <进程ID>
```

---

### 问题 5: Clojure 找不到

**错误信息**:
```
'clojure' 不是内部或外部命令
```

**解决方案**: 安装 Clojure

**使用 Scoop** (推荐):
```powershell
# 在 PowerShell 中
scoop install clojure
```

**或手动安装**: 访问 https://clojure.org/guides/install_clojure

---

### 问题 6: 依赖安装慢或失败

**解决方案**: 使用国内镜像

```bash
# 设置 Yarn 镜像
yarn config set registry https://registry.npmmirror.com

# 清理缓存
yarn cache clean

# 重新安装
rm -rf node_modules
yarn install --ignore-scripts
```

---

### 问题 7: Git 仓库警告

**警告信息**:
```
fatal: not a git repository (or any of the parent directories): .git
```

**原因**: Husky 尝试初始化 Git hooks,但找不到 `.git` 目录

**解决方案**: 
- 如果是从 GitHub 克隆的,确保 `.git` 目录存在
- 如果只是下载的代码包,可以忽略此警告 (不影响运行)
- 或者初始化 Git: `git init`

---

## 🔍 故障排查

### 检查环境

```bash
# 检查 Node 版本
node --version  # 应该 >= 22

# 检查 Yarn 版本
yarn --version  # 应该 ^1.22.22

# 检查 Java 版本
java --version  # 应该 >= 22

# 检查 Clojure 版本
clojure --version
```

### 检查端口占用

```bash
# 检查 3000 端口 (后端)
netstat -ano | grep 3000

# 检查 8080 端口 (前端)
netstat -ano | grep 8080

# 检查 6006 端口 (Storybook)
netstat -ano | grep 6006
```

### 清理并重新安装

```bash
# 清理 Node 依赖
rm -rf node_modules
yarn cache clean

# 清理编译产物
rm -rf target
rm -rf resources/frontend_client/app/dist

# 重新安装
yarn install --ignore-scripts
```

### 查看日志

前后端启动时会输出详细日志,注意查看:

- ✅ **成功标志**: `compiled successfully`, `Initialization COMPLETE`
- ❌ **错误标志**: `ERROR`, `FAILED`, `Exception`

---

## 📝 启动脚本

创建 `start.sh` 方便启动:

```bash
#!/bin/bash

echo "🚀 启动 Metabase 开发环境"
echo ""
echo "请按照以下步骤操作:"
echo ""
echo "1️⃣  在当前窗口启动前端:"
echo "   yarn build-hot:js"
echo ""
echo "2️⃣  打开新窗口启动后端:"
echo "   clojure -M:run"
echo ""
echo "3️⃣  等待两个服务都启动完成后,访问:"
echo "   http://localhost:3000"
echo ""
read -p "按 Enter 启动前端..."

yarn build-hot:js
```

使用:
```bash
chmod +x start.sh
./start.sh
```

---

## 🎯 最佳实践

### 1. 使用多个终端窗口

- **窗口 1**: 前端 (保持运行,查看前端日志)
- **窗口 2**: 后端 (保持运行,查看后端日志)
- **窗口 3**: 临时命令 (运行其他命令)

### 2. 监控资源使用

Metabase 开发环境需要较多资源:
- **内存**: 建议 8GB+
- **CPU**: 多核心更好
- **磁盘**: 至少 5GB 空间

### 3. 定期更新依赖

```bash
# 更新 Node 依赖
yarn upgrade

# 更新 Clojure 依赖
clojure -M:outdated
```

### 4. 使用 IDE 集成终端

在 VS Code 中可以同时打开多个终端,方便管理:
- `Ctrl + Shift + `` ` (打开终端)
- `Ctrl + Shift + 5` (分割终端)

---

## 📚 相关文档

- [项目总览](./00-项目总览.md)
- [项目概览与技术栈](./01-项目概览与技术栈.md)
- [项目架构与核心模块](./02-项目架构与核心模块.md)
- [开发工作流与最佳实践](./03-开发工作流与最佳实践.md)

---

## 💡 小贴士

1. **首次启动慢**: 正常现象,需要下载依赖和编译代码
2. **热重载**: 修改代码后会自动刷新,无需重启
3. **保存日志**: 遇到问题时,保存完整的错误日志方便排查
4. **定期清理**: `target/` 和 `node_modules/` 可以安全删除重建

---

**祝开发顺利! 🎉**
