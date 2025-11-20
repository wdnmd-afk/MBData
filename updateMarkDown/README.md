# Metabase 项目深度分析文档

> 完整的 Metabase 开源项目分析报告  
> 生成时间: 2025-11-19  
> 项目路径: e:\github\metabase

---

## 📚 文档索引

### [第一部分: 项目概览与技术栈](./01-项目概览与技术栈.md)
- 项目基本信息
- 核心特性
- 完整技术栈 (后端/前端/ClojureScript)
- 开发环境配置
- 快速启动指南

### [第二部分: 项目架构与核心模块](./02-项目架构与核心模块.md)
- 系统架构设计
- 查询处理器
- 驱动系统
- API 层
- 权限系统
- 数据模型
- 前端架构
- 嵌入式 SDK

### [第三部分: 开发工作流与最佳实践](./03-开发工作流与最佳实践.md)
- REPL 驱动开发
- 前后端开发流程
- 代码规范
- 性能优化
- 测试实践
- 调试技巧

### [第四部分: 测试体系与部署](./04-测试体系与部署.md)
- 单元测试 (Jest/Clojure)
- E2E 测试 (Cypress)
- 视觉测试 (Loki)
- 构建流程
- Docker 部署

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
