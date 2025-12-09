# Git 推送大仓库问题解决方案

## 问题描述

推送 Metabase 仓库到 GitHub 时出现错误:
```
error: unable to rewind rpc post data - try increasing http.postBuffer
```

**数据量**:
- 对象数: 18,199 个
- 总大小: 89.50 MiB
- 传输速度: 3.79 MiB/s

---

## 解决方案

### 方案 1: 增加 HTTP 缓冲区(推荐)

```bash
# 1. 先取消当前卡住的推送(按 Ctrl+C)

# 2. 增加 HTTP POST 缓冲区到 500MB
git config --global http.postBuffer 524288000

# 3. 重新推送
git push -u origin main
```

**说明**:
- `524288000` = 500MB (500 * 1024 * 1024)
- `--global` 表示全局设置,对所有仓库生效
- 如果只想对当前仓库设置,去掉 `--global`

---

### 方案 2: 使用 SSH 协议(更稳定)

HTTP 协议在推送大文件时容易出问题,SSH 更稳定。

#### 步骤 1: 检查是否有 SSH 密钥

```bash
ls -la ~/.ssh
```

如果看到 `id_rsa` 和 `id_rsa.pub`,说明已有密钥,跳到步骤 3。

#### 步骤 2: 生成 SSH 密钥(如果没有)

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 按提示操作:
# - 按 Enter 使用默认位置
# - 输入密码(可选,可以留空)
```

#### 步骤 3: 复制公钥

```bash
# Windows (Git Bash)
cat ~/.ssh/id_rsa.pub

# 或者用 clip 命令直接复制到剪贴板
clip < ~/.ssh/id_rsa.pub
```

#### 步骤 4: 添加到 GitHub

1. 登录 GitHub
2. 点击右上角头像 → **Settings**
3. 左侧菜单 → **SSH and GPG keys**
4. 点击 **New SSH key**
5. 粘贴公钥内容
6. 点击 **Add SSH key**

#### 步骤 5: 更改远程仓库 URL

```bash
# 查看当前远程仓库
git remote -v

# 如果是 HTTPS URL (类似 https://github.com/username/repo.git)
# 改为 SSH URL
git remote set-url origin git@github.com:username/metabase.git

# 替换 username 为你的 GitHub 用户名
```

#### 步骤 6: 推送

```bash
git push -u origin main
```

---

### 方案 3: 分批推送(如果上述方案都失败)

如果仓库太大,可以分批推送历史提交。

```bash
# 1. 查看提交历史
git log --oneline

# 2. 先推送部分提交(假设最近 10 个)
git push origin HEAD~10:refs/heads/main

# 3. 然后推送剩余的
git push -u origin main
```

---

### 方案 4: 使用 Git LFS(如果有大文件)

如果仓库中有大型二进制文件,应该使用 Git LFS。

```bash
# 1. 安装 Git LFS
git lfs install

# 2. 跟踪大文件类型(例如)
git lfs track "*.jar"
git lfs track "*.db"
git lfs track "*.zip"

# 3. 提交 .gitattributes
git add .gitattributes
git commit -m "Configure Git LFS"

# 4. 推送
git push -u origin main
```

---

## 检查是否有不应该提交的文件

### 常见问题文件

Metabase 项目中,以下文件**不应该**提交到 Git:

```bash
# 检查是否误提交了这些文件
git ls-files | grep -E "(metabase\.db|node_modules|target|\.class$)"
```

**不应该提交的文件**:
- `metabase.db.mv.db` - 数据库文件
- `metabase.db.trace.db` - 数据库跟踪文件
- `node_modules/` - Node.js 依赖
- `target/` - 编译输出
- `*.class` - Java 编译文件
- `.cpcache/` - Clojure 缓存

### 如果误提交了大文件

```bash
# 1. 从 Git 历史中删除文件(但保留在工作目录)
git rm --cached metabase.db.mv.db
git rm --cached metabase.db.trace.db

# 2. 确保 .gitignore 包含这些文件
echo "metabase.db.mv.db" >> .gitignore
echo "metabase.db.trace.db" >> .gitignore

# 3. 提交更改
git add .gitignore
git commit -m "Remove database files from Git tracking"

# 4. 如果已经推送过,需要强制推送(危险!)
# git push -f origin main
```

---

## 推荐的 .gitignore 配置

确保你的 `.gitignore` 包含以下内容:

```gitignore
# Metabase 数据库文件
metabase.db.mv.db
metabase.db.trace.db
*.db
*.db.mv.db
*.db.trace.db

# 编译输出
target/
.cpcache/
*.class

# Node.js
node_modules/
npm-debug.log
yarn-error.log

# IDE
.idea/
*.iml
.vscode/
*.swp
*.swo

# 操作系统
.DS_Store
Thumbs.db

# 临时文件
*.log
*.tmp
*.bak
```

---

## 优化 Git 配置

```bash
# 增加网络超时时间
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999

# 增加 POST 缓冲区
git config --global http.postBuffer 524288000

# 启用压缩(减少传输数据量)
git config --global core.compression 9

# 查看所有配置
git config --list
```

---

## 故障排查步骤

### 1. 检查网络连接

```bash
# 测试 GitHub 连接
ping github.com

# 测试 SSH 连接(如果使用 SSH)
ssh -T git@github.com
```

### 2. 检查仓库大小

```bash
# 查看仓库大小
du -sh .git

# 查看最大的文件
git rev-list --objects --all | \
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
  sed -n 's/^blob //p' | \
  sort --numeric-sort --key=2 | \
  tail -20
```

### 3. 清理不必要的文件

```bash
# 清理未跟踪的文件
git clean -fd

# 清理 Git 缓存
git gc --aggressive --prune=now
```

---

## 当前推送卡住的处理

### 立即操作

1. **按 `Ctrl+C` 取消当前推送**

2. **增加缓冲区**:
   ```bash
   git config --global http.postBuffer 524288000
   ```

3. **重新推送**:
   ```bash
   git push -u origin main
   ```

### 如果还是失败

尝试 SSH 方式:
```bash
# 改用 SSH
git remote set-url origin git@github.com:你的用户名/metabase.git

# 推送
git push -u origin main
```

---

## 预防措施

### 1. 使用 .gitignore

确保在第一次提交前配置好 `.gitignore`,避免提交不必要的文件。

### 2. 定期清理

```bash
# 定期运行垃圾回收
git gc --auto
```

### 3. 使用 Git LFS

对于大型二进制文件(>50MB),使用 Git LFS:
```bash
git lfs track "*.jar"
git lfs track "*.db"
```

### 4. 检查提交前的大小

```bash
# 查看即将提交的文件大小
git diff --cached --stat
```

---

## 常见错误及解决方案

### 错误 1: RPC failed; HTTP 413

**原因**: 推送的数据超过服务器限制

**解决**:
```bash
git config --global http.postBuffer 524288000
```

### 错误 2: Connection reset by peer

**原因**: 网络不稳定

**解决**:
```bash
# 使用 SSH 代替 HTTPS
git remote set-url origin git@github.com:username/repo.git
```

### 错误 3: fatal: The remote end hung up unexpectedly

**原因**: 推送超时

**解决**:
```bash
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
```

---

## 总结

### 快速修复命令

```bash
# 1. 取消当前推送(Ctrl+C)

# 2. 增加缓冲区
git config --global http.postBuffer 524288000

# 3. 重新推送
git push -u origin main
```

### 长期解决方案

1. ✅ 使用 SSH 协议代替 HTTPS
2. ✅ 配置正确的 .gitignore
3. ✅ 不要提交数据库文件和编译输出
4. ✅ 对大文件使用 Git LFS

### 检查清单

- [ ] 已增加 `http.postBuffer`
- [ ] 已配置 SSH 密钥
- [ ] 已检查 `.gitignore`
- [ ] 已确认没有提交数据库文件
- [ ] 已清理不必要的文件

---

## 参考资源

- [Git 官方文档 - HTTP 配置](https://git-scm.com/docs/git-config#Documentation/git-config.txt-httppostBuffer)
- [GitHub 文档 - 推送大文件](https://docs.github.com/en/repositories/working-with-files/managing-large-files)
- [Git LFS 官网](https://git-lfs.github.com/)
