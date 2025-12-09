# Metabase 项目完整文档

> 从零到运行的完整指南，包含所有问题解决方案  
> 更新时间: 2025-11-20  
> 项目路径: e:\github\metabase

---

## 🚀 快速开始

### 新手必读

**👉 [完整部署指南](./00-Metabase完整部署指南.md)** ⭐⭐⭐

这是最重要的文档！包含：
- ✅ 从克隆项目到成功运行的完整流程
- ✅ 每个步骤的详细说明
- ✅ 所有可能遇到的问题和解决方案
- ✅ 12+ 个常见错误的修复方法

**建议**: 先阅读这份文档，按照步骤操作，遇到问题时查看对应的解决方案。

---

## 📚 核心文档

### [00-项目总览](./00-项目总览.md)
- 项目简介和特色
- 文档结构导航
- 核心技术栈概览
- 快速命令参考

### [01-项目概览与技术栈](./01-项目概览与技术栈.md)
- 项目基本信息
- 核心特性
- 完整技术栈 (后端/前端/ClojureScript)
- 开发环境配置
- 快速启动指南

### [02-项目架构与核心模块](./02-项目架构与核心模块.md)
- 系统架构设计
- 查询处理器
- 驱动系统
- API 层
- 权限系统
- 数据模型
- 前端架构
- 嵌入式 SDK

### [03-开发工作流与最佳实践](./03-开发工作流与最佳实践.md)
- REPL 驱动开发
- 前后端开发流程
- 代码规范
- 性能优化
- 测试实践
- 调试技巧

---

## 🔧 问题解决文档

### [04-国际化问题诊断与修复](./04-国际化问题诊断与修复.md)
**问题**: 设置中文后系统没有正确变为中文

**解决方案**:
- 编译翻译文件: `./bin/i18n/build-translation-resources`
- 验证翻译文件是否生成
- 重启后端和前端

### [05-数据存储位置说明](./05-数据存储位置说明.md)
**问题**: 不知道数据存储在哪里，如何备份

**内容**:
- 数据库文件位置: `metabase.db.mv.db`
- 包含的数据: 用户、数据源、仪表板等
- 备份和恢复方法
- 迁移到生产数据库(PostgreSQL/MySQL)

### [06-Git推送大仓库问题解决](./06-Git推送大仓库问题解决.md)
**问题**: Git 推送时卡住，显示 "unable to rewind rpc post data"

**解决方案**:
- 增加 HTTP 缓冲区: `git config --global http.postBuffer 524288000`
- 使用 SSH 代替 HTTPS
- 检查是否误提交大文件

### [07-删除企业版功能指南](./07-删除企业版功能指南.md)
**问题**: 想要删除或隐藏企业版相关功能和 UI

**内容**:
- ⚠️ 为什么不建议完全删除
- 企业版功能概览(525+ 后端文件, 1402+ 前端文件)
- 多种删除/隐藏方案对比
- **推荐**: 使用 CSS 隐藏企业版 UI(30分钟完成)
- 快速实施脚本: `./hide-enterprise-ui.sh`

### [08-隐藏企业版推销组件](./08-隐藏企业版推销组件.md) ⭐
**问题**: 隐藏"更高级的嵌入"、"嵌入式 React 分析 SDK"等推销信息

**快速使用**:
```bash
chmod +x hide-enterprise-upsells.sh
./hide-enterprise-upsells.sh
yarn build-hot
```

**隐藏内容**:
- ✅ 更高级的嵌入 (More advanced embeds)
- ✅ 嵌入式 React 分析 SDK
- ✅ "Try Metabase Pro" 按钮
- ✅ SSO、白标、高级权限等推销
- ✅ 所有 19 个企业版推销组件

### [Docker 一键部署](../DOCKER-DEPLOY.md) 🐳
**问题**: 如何使用 Docker 快速部署 Metabase 项目

**三步部署**:
```bash
# 1. 检查环境
chmod +x check.sh && ./check.sh

# 2. 修复问题(如果需要)
chmod +x fix.sh && ./fix.sh

# 3. 一键部署
chmod +x deploy.sh && ./deploy.sh
```

**三个核心脚本**:
- ✅ `check.sh` - 环境检查(Docker/网络/端口)
- ✅ `fix.sh` - 自动修复(镜像加速器/Docker启动/网络)
- ✅ `deploy.sh` - 一键部署(快速部署/完整构建/日志查看)

**访问地址**: http://localhost:3000

**常见问题自动解决**:
- ✅ Docker 未运行 → `./fix.sh` 选项 2
- ✅ 镜像下载慢 → `./fix.sh` 选项 1
- ✅ 网络超时 → `./fix.sh` 选项 5
- ✅ 一键修复所有 → `./fix.sh` 选项 6

---

## 🎯 快速查找

### 开始开发
→ [快速启动](./01-项目概览与技术栈.md#-快速启动)

### 了解技术栈
→ [技术栈分析](./01-项目概览与技术栈.md#-技术栈分析)

### 查看架构
→ [系统架构](./02-项目架构与核心模块.md#-整体架构)

### 编写代码
→ [开发工作流](./03-开发工作流与最佳实践.md#-开发工作流)

---

## 📊 核心信息

**技术栈**: Clojure + React + TypeScript  
**数据库**: 支持 15+ 种数据库  
**测试**: Jest + Cypress + Clojure Test  
**构建**: Rspack + Shadow-CLJS  

**常用命令**:
```bash
yarn dev              # 启动开发环境
yarn test-unit        # 前端测试
clojure -X:dev:test   # 后端测试
./bin/mage kondo      # 代码检查
```

---

## 🔗 相关资源

- [Metabase 官网](https://www.metabase.com)
- [GitHub](https://github.com/metabase/metabase)
- [开发者文档](https://www.metabase.com/docs/latest/developers-guide/)
- [CLAUDE.md](../CLAUDE.md) - AI 开发指南
