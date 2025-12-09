# Metabase 完整部署指南 - 从零到运行

> 本文档记录了从拉取 Metabase 项目到成功运行的完整过程,包含所有遇到的问题和详细解决方案。

## 目录

- [环境要求](#环境要求)
- [步骤 1: 克隆项目](#步骤-1-克隆项目)
- [步骤 2: 安装依赖](#步骤-2-安装依赖)
- [步骤 3: 编译翻译文件](#步骤-3-编译翻译文件)
- [步骤 4: 启动后端](#步骤-4-启动后端)
- [步骤 5: 启动前端](#步骤-5-启动前端)
- [步骤 6: 初始化设置](#步骤-6-初始化设置)
- [常见问题汇总](#常见问题汇总)
- [Git 推送问题](#git-推送问题)

---

## 环境要求

### 必需软件

| 软件 | 版本要求 | 用途 |
|------|---------|------|
| **Node.js** | >= 22.x | 前端构建和运行 |
| **Yarn** | ^1.12.3 | 前端依赖管理 |
| **Java JDK** | >= 11 | 后端运行 |
| **Clojure CLI** | 最新版 | 后端构建和运行 |
| **Git** | 最新版 | 版本控制 |

### 验证环境

```bash
# 检查 Node.js 版本
node --version
# 应该显示: v22.x.x 或更高

# 检查 Yarn 版本
yarn --version
# 应该显示: 1.x.x

# 检查 Java 版本
java -version
# 应该显示: 11 或更高

# 检查 Clojure CLI
clojure --version
# 应该显示: Clojure CLI version 1.x.x

# 检查 Git
git --version
```

---

## 步骤 1: 克隆项目

### 1.1 克隆仓库

```bash
# 克隆官方仓库
git clone https://github.com/metabase/metabase.git
cd metabase

# 或者克隆你的 fork
git clone https://github.com/你的用户名/metabase.git
cd metabase
```

### 1.2 检查项目结构

```bash
# 查看项目结构
ls -la

# 应该看到:
# - frontend/        前端代码
# - src/             后端代码
# - resources/       资源文件
# - locales/         翻译文件
# - package.json     前端依赖配置
# - deps.edn         后端依赖配置
```

---

## 步骤 2: 安装依赖

### 2.1 安装前端依赖

```bash
# 安装 Node.js 依赖
yarn install
```

**预计耗时**: 5-10 分钟(取决于网络速度)

#### ⚠️ 可能遇到的问题 1: Yarn 安装失败

**错误信息**:
```
error An unexpected error occurred: "ENOENT: no such file or directory"
```

**解决方案**:
```bash
# 清理缓存
yarn cache clean

# 删除 node_modules
rm -rf node_modules

# 重新安装
yarn install
```

#### ⚠️ 可能遇到的问题 2: 网络超时

**错误信息**:
```
error An unexpected error occurred: "https://registry.yarnpkg.com/...: ETIMEDOUT"
```

**解决方案**:
```bash
# 使用淘宝镜像
yarn config set registry https://registry.npmmirror.com

# 重新安装
yarn install
```

### 2.2 下载 Clojure 依赖

```bash
# Clojure 依赖会在首次运行时自动下载
# 可以提前下载以节省时间
clojure -P -X:dev
```

**预计耗时**: 3-5 分钟

---

## 步骤 3: 编译翻译文件

### 3.1 为什么需要这一步?

Metabase 的翻译文件存储为 `.po` 格式,需要编译成 `.json` 格式才能被前端加载。

**如果跳过这一步**: 切换语言功能将不工作,界面会保持英文。

### 3.2 编译翻译资源

```bash
# 在项目根目录执行
./bin/i18n/build-translation-resources
```

**预计耗时**: 10-30 秒

**输出示例**:
```
Create frontend artifact resources/frontend_client/app/locales/zh-CN.json
  Wrote 5663 messages.
Artifacts for locale "zh-CN" created successfully.
Translation resources built successfully.
```

### 3.3 验证翻译文件

```bash
# 检查是否生成了中文翻译文件
ls -la resources/frontend_client/app/locales/zh*.json

# 应该看到:
# zh_CN.json (约 454 KB)
# zh_TW.json (约 455 KB)
# zh_HK.json (约 454 KB)
```

#### ⚠️ 可能遇到的问题 3: 权限错误

**错误信息**:
```
bash: ./bin/i18n/build-translation-resources: Permission denied
```

**解决方案**:
```bash
# 添加执行权限
chmod +x ./bin/i18n/build-translation-resources

# 重新执行
./bin/i18n/build-translation-resources
```

#### ⚠️ 可能遇到的问题 4: Clojure CLI 未安装

**错误信息**:
```
clojure: command not found
```

**解决方案**:

**Windows**:
```powershell
# 使用 Scoop 安装
scoop install clojure

# 或下载安装包
# https://github.com/clojure/brew-install/releases
```

**macOS**:
```bash
brew install clojure/tools/clojure
```

**Linux**:
```bash
curl -O https://download.clojure.org/install/linux-install-1.11.1.1435.sh
chmod +x linux-install-1.11.1.1435.sh
sudo ./linux-install-1.11.1.1435.sh
```

---

## 步骤 4: 启动后端

### 4.1 首次启动

```bash
# 在项目根目录执行
clojure -M:run
```

**预计耗时**: 
- 首次启动: 2-5 分钟(需要下载依赖和初始化数据库)
- 后续启动: 30-60 秒

### 4.2 启动成功的标志

看到以下日志表示启动成功:

```
2025-11-20 08:23:28,373 INFO core.core :: Metabase Initialization COMPLETE
2025-11-20 08:23:28,374 INFO core.core :: Metabase v0.51.0 is RUNNING
```

**访问地址**: http://localhost:3000

### 4.3 数据库初始化

首次启动时,Metabase 会自动创建 H2 数据库:

```
metabase.db.mv.db       # 主数据库文件
metabase.db.trace.db    # 跟踪日志文件
```

**位置**: 项目根目录

**大小**: 初始约 1-2 MB

#### ⚠️ 可能遇到的问题 5: 端口被占用

**错误信息**:
```
java.net.BindException: Address already in use
```

**解决方案**:

**方法 1: 更改端口**
```bash
# 使用环境变量指定端口
export MB_JETTY_PORT=3001
clojure -M:run
```

**方法 2: 关闭占用端口的进程**
```bash
# Windows
netstat -ano | findstr :3000
taskkill /PID <进程ID> /F

# macOS/Linux
lsof -ti:3000 | xargs kill -9
```

#### ⚠️ 可能遇到的问题 6: 数据库初始化失败

**错误信息**:
```
Unique index or primary key violation: "PRIMARY KEY ON PUBLIC.COLLECTION(ID)"
```

**原因**: 数据库文件损坏或版本不兼容

**解决方案**:
```bash
# 1. 停止后端(Ctrl+C)

# 2. 备份现有数据库(如果需要)
cp metabase.db.mv.db metabase.db.mv.db.backup

# 3. 删除数据库文件
rm metabase.db.mv.db
rm metabase.db.trace.db

# 4. 重新启动(会创建新数据库)
clojure -M:run
```

#### ⚠️ 可能遇到的问题 7: 内存不足

**错误信息**:
```
java.lang.OutOfMemoryError: Java heap space
```

**解决方案**:
```bash
# 增加 JVM 堆内存
export JVM_OPTS="-Xmx2g"
clojure -M:run
```

---

## 步骤 5: 启动前端

### 5.1 开发模式启动

**打开新的终端窗口**,执行:

```bash
cd /path/to/metabase
yarn build-hot
```

**预计耗时**:
- 首次编译: 2-3 分钟
- 后续热更新: 几秒钟

### 5.2 编译成功的标志

看到以下输出表示编译成功:

```
webpack compiled successfully in 120000ms
```

**访问地址**: http://localhost:3000 (与后端相同)

### 5.3 前端热更新

修改前端代码后,Webpack 会自动重新编译并刷新浏览器。

#### ⚠️ 可能遇到的问题 8: 编译错误

**错误信息**:
```
Module not found: Error: Can't resolve 'xxx'
```

**解决方案**:
```bash
# 1. 清理缓存
rm -rf node_modules/.cache

# 2. 重新安装依赖
yarn install

# 3. 重新编译
yarn build-hot
```

#### ⚠️ 可能遇到的问题 9: 端口冲突

**错误信息**:
```
Port 3000 is already in use
```

**说明**: 这是正常的,前端和后端共用 3000 端口。确保后端已启动即可。

#### ⚠️ 可能遇到的问题 10: TypeScript 错误

**错误信息**:
```
TS2304: Cannot find name 'xxx'
```

**解决方案**:
```bash
# 重新生成类型定义
yarn build-types

# 重新编译
yarn build-hot
```

---

## 步骤 6: 初始化设置

### 6.1 首次访问

打开浏览器访问: http://localhost:3000

会自动跳转到设置向导。

### 6.2 设置步骤

#### 第 1 步: 选择语言

- 选择 **简体中文** 或保持 **English**
- 点击 **Next**

#### 第 2 步: 创建管理员账户

填写以下信息:
- **First name**: 你的名字
- **Last name**: 你的姓氏
- **Email**: 管理员邮箱
- **Password**: 设置密码(至少 6 位)

点击 **Next**

#### 第 3 步: 添加数据库(可选)

- 可以跳过,稍后添加
- 或者添加测试数据库

点击 **Next** 或 **Skip**

#### 第 4 步: 使用偏好

选择使用场景,点击 **Next**

#### 第 5 步: 完成

点击 **Take me to Metabase**

### 6.3 验证安装

登录后应该看到:
- ✅ 主仪表板页面
- ✅ 顶部导航栏
- ✅ 左侧菜单
- ✅ 右上角用户头像

#### ⚠️ 可能遇到的问题 11: 设置向导无法访问

**现象**: 浏览器显示 "无法访问此网站"

**检查清单**:
```bash
# 1. 确认后端正在运行
# 在后端终端查看是否有 "Metabase is RUNNING" 日志

# 2. 确认端口正确
curl http://localhost:3000/api/health
# 应该返回: {"status":"ok"}

# 3. 检查防火墙
# 确保 3000 端口未被防火墙阻止
```

#### ⚠️ 可能遇到的问题 12: 语言切换不生效

**现象**: 选择中文后界面仍是英文

**原因**: 翻译文件未编译

**解决方案**:
```bash
# 1. 停止前端和后端(Ctrl+C)

# 2. 编译翻译文件
./bin/i18n/build-translation-resources

# 3. 重启后端
clojure -M:run

# 4. 重启前端(新终端)
yarn build-hot

# 5. 清除浏览器缓存并刷新
```

---

## 常见问题汇总

### 问题 1: 如何停止服务?

**停止后端**:
```bash
# 在运行 clojure -M:run 的终端按 Ctrl+C
```

**停止前端**:
```bash
# 在运行 yarn build-hot 的终端按 Ctrl+C
```

### 问题 2: 如何重启服务?

```bash
# 后端
clojure -M:run

# 前端(新终端)
yarn build-hot
```

### 问题 3: 数据存储在哪里?

**数据库文件**: `metabase.db.mv.db` (项目根目录)

**包含内容**:
- 用户账户
- 数据库连接配置
- 仪表板和问题
- 所有设置

**备份方法**:
```bash
# 停止后端后执行
cp metabase.db.mv.db metabase.db.mv.db.backup
```

### 问题 4: 如何添加数据库连接?

1. 登录 Metabase
2. 点击右上角齿轮图标 → **Admin Settings**
3. 左侧菜单 → **Databases**
4. 点击 **Add database**
5. 选择数据库类型并填写连接信息
6. 点击 **Save**

### 问题 5: 如何更改语言?

1. 点击右上角头像
2. 选择 **Account Settings**
3. 在 **Language** 下拉框选择语言
4. 点击 **Update**
5. 页面会自动刷新

### 问题 6: 忘记管理员密码怎么办?

```bash
# 使用命令行重置密码
clojure -M:run reset-password admin@example.com
```

### 问题 7: 如何查看日志?

**后端日志**: 直接在运行 `clojure -M:run` 的终端查看

**前端日志**: 
- 浏览器开发者工具(F12) → Console 标签
- Webpack 编译日志在运行 `yarn build-hot` 的终端

### 问题 8: 如何清理缓存?

```bash
# 清理前端缓存
rm -rf node_modules/.cache
rm -rf .webpack

# 清理 Clojure 缓存
rm -rf .cpcache
rm -rf target

# 清理 Yarn 缓存
yarn cache clean
```

### 问题 9: 如何更新依赖?

```bash
# 更新前端依赖
yarn upgrade

# 更新 Clojure 依赖
# 编辑 deps.edn 文件,修改版本号
```

### 问题 10: 如何运行测试?

```bash
# 前端测试
yarn test

# 后端测试
clojure -X:dev:test
```

---

## Git 推送问题

### 问题: 推送大仓库时卡住

**错误信息**:
```
error: unable to rewind rpc post data - try increasing http.postBuffer
```

**原因**: Git HTTP 缓冲区太小

**解决方案**:

#### 方法 1: 增加缓冲区

```bash
# 1. 取消当前推送(Ctrl+C)

# 2. 增加缓冲区到 500MB
git config --global http.postBuffer 524288000

# 3. 重新推送
git push -u origin main
```

#### 方法 2: 使用 SSH(推荐)

```bash
# 1. 生成 SSH 密钥(如果没有)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 2. 复制公钥
cat ~/.ssh/id_rsa.pub

# 3. 添加到 GitHub
# Settings → SSH and GPG keys → New SSH key

# 4. 更改远程仓库 URL
git remote set-url origin git@github.com:username/metabase.git

# 5. 推送
git push -u origin main
```

### 检查是否误提交大文件

```bash
# 检查是否提交了数据库文件
git ls-files | grep -E "\.db$"

# 如果看到 metabase.db.mv.db,需要移除
git rm --cached metabase.db.mv.db
git rm --cached metabase.db.trace.db

# 添加到 .gitignore
echo "metabase.db.mv.db" >> .gitignore
echo "metabase.db.trace.db" >> .gitignore

# 提交
git add .gitignore
git commit -m "Remove database files from Git tracking"
```

---

## 开发工作流

### 日常开发流程

```bash
# 1. 启动后端(终端 1)
cd /path/to/metabase
clojure -M:run

# 2. 启动前端(终端 2)
cd /path/to/metabase
yarn build-hot

# 3. 开始开发
# - 修改代码
# - 保存文件
# - 浏览器自动刷新(前端)
# - 后端需要重启才能看到更改

# 4. 提交代码
git add .
git commit -m "描述你的更改"
git push
```

### 代码规范检查

```bash
# 前端代码检查
yarn lint

# 前端代码格式化
yarn format

# 后端代码检查
clojure -M:eastwood
```

---

## 性能优化建议

### 1. 增加 JVM 内存

```bash
export JVM_OPTS="-Xmx4g -Xms2g"
clojure -M:run
```

### 2. 使用 SSD

将项目放在 SSD 上可以显著提升编译速度。

### 3. 关闭不必要的服务

开发时关闭其他占用内存的应用。

### 4. 使用增量编译

前端已默认启用热更新,无需额外配置。

---

## 故障排查流程

### 步骤 1: 检查环境

```bash
node --version    # >= 22
yarn --version    # >= 1.12
java -version     # >= 11
clojure --version # 最新版
```

### 步骤 2: 检查依赖

```bash
# 前端依赖
ls node_modules | wc -l
# 应该有数百个包

# 后端依赖
ls ~/.m2/repository | wc -l
# 应该有大量 JAR 文件
```

### 步骤 3: 检查端口

```bash
# 检查 3000 端口
curl http://localhost:3000/api/health

# 应该返回: {"status":"ok"}
```

### 步骤 4: 查看日志

- 后端日志: 运行 `clojure -M:run` 的终端
- 前端日志: 运行 `yarn build-hot` 的终端
- 浏览器日志: F12 → Console

### 步骤 5: 清理重试

```bash
# 清理所有缓存
rm -rf node_modules/.cache
rm -rf .cpcache
rm -rf target

# 重新安装
yarn install

# 重新启动
clojure -M:run  # 终端 1
yarn build-hot  # 终端 2
```

---

## 生产部署建议

### 不要在生产环境使用 H2 数据库

**原因**:
- ❌ 不支持多实例
- ❌ 性能有限
- ❌ 容易损坏

**推荐**: PostgreSQL 或 MySQL

### 迁移到 PostgreSQL

```bash
# 1. 创建 PostgreSQL 数据库
createdb metabase

# 2. 设置环境变量
export MB_DB_TYPE=postgres
export MB_DB_DBNAME=metabase
export MB_DB_PORT=5432
export MB_DB_USER=metabase
export MB_DB_PASS=your_password
export MB_DB_HOST=localhost

# 3. 从 H2 迁移数据
clojure -M:run load-from-h2

# 4. 启动(使用 PostgreSQL)
clojure -M:run
```

---

## 快速参考

### 常用命令

```bash
# 启动后端
clojure -M:run

# 启动前端
yarn build-hot

# 编译翻译
./bin/i18n/build-translation-resources

# 安装依赖
yarn install

# 清理缓存
yarn cache clean
rm -rf .cpcache

# 运行测试
yarn test
clojure -X:dev:test

# 代码检查
yarn lint

# 备份数据库
cp metabase.db.mv.db metabase.db.mv.db.backup
```

### 重要文件

```
metabase/
├── frontend/              # 前端代码
├── src/                   # 后端代码
├── resources/             # 资源文件
├── locales/               # 翻译源文件
├── metabase.db.mv.db      # 数据库文件(不要提交)
├── package.json           # 前端依赖
├── deps.edn               # 后端依赖
└── .gitignore             # Git 忽略配置
```

### 重要端口

- **3000**: Metabase 主服务(前端+后端)
- **8082**: H2 Console(如果启动)

### 环境变量

```bash
# 数据库配置
MB_DB_TYPE=h2              # 数据库类型
MB_DB_FILE=metabase.db     # H2 数据库文件

# 服务器配置
MB_JETTY_PORT=3000         # 服务端口
MB_JETTY_HOST=0.0.0.0      # 监听地址

# JVM 配置
JVM_OPTS="-Xmx2g"          # 堆内存大小
```

---

## 总结

### 完整启动流程

```bash
# 1. 克隆项目
git clone https://github.com/metabase/metabase.git
cd metabase

# 2. 安装依赖
yarn install

# 3. 编译翻译
./bin/i18n/build-translation-resources

# 4. 启动后端(终端 1)
clojure -M:run

# 5. 启动前端(终端 2)
yarn build-hot

# 6. 访问
# http://localhost:3000
```

### 关键检查点

- ✅ Node.js >= 22
- ✅ Yarn 已安装
- ✅ Java >= 11
- ✅ Clojure CLI 已安装
- ✅ 翻译文件已编译
- ✅ 后端显示 "Metabase is RUNNING"
- ✅ 前端显示 "webpack compiled successfully"
- ✅ 浏览器可以访问 http://localhost:3000

### 遇到问题时

1. 查看本文档的"常见问题汇总"
2. 检查终端的错误日志
3. 清理缓存后重试
4. 查看浏览器控制台(F12)
5. 参考详细文档: `updateMarkDown/` 目录

---

## 相关文档

- [01-项目概览与技术栈.md](./01-项目概览与技术栈.md)
- [02-项目架构与核心模块.md](./02-项目架构与核心模块.md)
- [03-开发工作流与最佳实践.md](./03-开发工作流与最佳实践.md)
- [04-国际化问题诊断与修复.md](./04-国际化问题诊断与修复.md)
- [05-数据存储位置说明.md](./05-数据存储位置说明.md)
- [06-Git推送大仓库问题解决.md](./06-Git推送大仓库问题解决.md)

---

## 更新日志

- **2025-11-20**: 初始版本,记录完整部署流程和所有遇到的问题

---

**祝你开发顺利! 🚀**
