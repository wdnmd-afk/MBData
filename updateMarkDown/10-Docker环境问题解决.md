# Docker 环境问题解决指南

> 解决 Docker Desktop 未启动、WSL 2 配置等常见问题

---

## 目录

- [问题诊断](#问题诊断)
- [Windows 系统解决方案](#windows-系统解决方案)
- [常见错误和解决方法](#常见错误和解决方法)
- [完整安装流程](#完整安装流程)
- [验证和测试](#验证和测试)

---

## 问题诊断

### 你遇到的错误

```
error during connect: Head "http://%2F%2F.%2Fpipe%2FdockerDesktopLinuxEngine/_ping": 
open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified.
```

**核心原因**: Docker Desktop 没有运行!

### 快速诊断

运行诊断脚本:

```bash
chmod +x check-docker.sh
./check-docker.sh
```

或手动检查:

```bash
# 检查 Docker 是否运行
docker ps

# 如果看到错误,说明 Docker 未运行
```

---

## Windows 系统解决方案

### 方案 1: 启动 Docker Desktop(最常见)

#### 步骤 1: 启动 Docker Desktop

1. **按 Windows 键**,搜索 "Docker Desktop"
2. **点击启动** Docker Desktop
3. **等待启动完成**(任务栏图标变为绿色)

**预计耗时**: 30-60 秒

#### 步骤 2: 验证启动

```bash
# 等待 Docker 完全启动
# 任务栏图标显示: Docker Desktop is running

# 验证
docker ps
```

**成功输出**:
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

#### 步骤 3: 重新运行部署

```bash
./docker-deploy.sh
# 选择选项 1
```

---

### 方案 2: WSL 2 未安装或配置错误

#### 检查 WSL 2 状态

```powershell
# 以管理员身份运行 PowerShell
wsl --list --verbose
```

**正常输出**:
```
  NAME                   STATE           VERSION
* Ubuntu                 Running         2
  docker-desktop         Running         2
  docker-desktop-data    Running         2
```

#### 安装/更新 WSL 2

```powershell
# 1. 安装 WSL 2
wsl --install

# 2. 更新 WSL
wsl --update

# 3. 设置默认版本为 WSL 2
wsl --set-default-version 2

# 4. 重启电脑
shutdown /r /t 0
```

#### 启用 WSL 功能

```powershell
# 以管理员身份运行 PowerShell

# 启用 WSL
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 启用虚拟机平台
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 重启电脑
shutdown /r /t 0
```

---

### 方案 3: 虚拟化未启用

#### 检查虚拟化状态

**方法 1: 任务管理器**

1. 按 `Ctrl + Shift + Esc` 打开任务管理器
2. 点击 "性能" 标签
3. 选择 "CPU"
4. 查看 "虚拟化" 状态

**应该显示**: 已启用

**方法 2: 命令行**

```powershell
# 以管理员身份运行
systeminfo | findstr /i "虚拟化"
```

**应该显示**: 
```
固件中已启用虚拟化: 是
```

#### 启用虚拟化

如果虚拟化未启用,需要在 BIOS 中启用:

**步骤**:

1. **重启电脑**
2. **进入 BIOS 设置**:
   - Dell: 按 `F2`
   - HP: 按 `F10`
   - Lenovo: 按 `F1` 或 `F2`
   - ASUS: 按 `F2` 或 `Del`
   - 其他: 通常是 `F2`, `F10`, `Del` 或 `Esc`

3. **找到虚拟化选项**:
   - Intel CPU: 找到 "Intel VT-x" 或 "Intel Virtualization Technology"
   - AMD CPU: 找到 "AMD-V" 或 "SVM Mode"

4. **启用虚拟化**:
   - 将选项改为 "Enabled"

5. **保存并退出**:
   - 按 `F10` 保存
   - 选择 "Yes" 确认

6. **重启电脑**

---

### 方案 4: Hyper-V 配置问题

#### 启用 Hyper-V(如果使用 Hyper-V 后端)

```powershell
# 以管理员身份运行 PowerShell

# 启用 Hyper-V
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All

# 重启电脑
shutdown /r /t 0
```

#### 或使用 DISM

```powershell
# 启用 Hyper-V
DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V

# 重启电脑
shutdown /r /t 0
```

---

### 方案 5: Docker Desktop 配置问题

#### 重置 Docker Desktop

1. **打开 Docker Desktop**
2. **点击设置图标**(右上角齿轮)
3. **选择 "Troubleshoot"**
4. **点击 "Reset to factory defaults"**
5. **确认重置**
6. **等待重置完成**
7. **重新启动 Docker Desktop**

#### 切换 WSL 2 后端

1. **打开 Docker Desktop**
2. **设置 -> General**
3. **勾选 "Use the WSL 2 based engine"**
4. **点击 "Apply & Restart"**

---

## 常见错误和解决方法

### 错误 1: Docker Desktop starting... 一直卡住

**原因**: WSL 2 或虚拟化问题

**解决方案**:

```powershell
# 1. 完全关闭 Docker Desktop
# 任务管理器 -> 结束 Docker Desktop 进程

# 2. 重启 WSL
wsl --shutdown

# 3. 重新启动 Docker Desktop
```

---

### 错误 2: WSL 2 installation is incomplete

**原因**: WSL 2 内核未安装

**解决方案**:

```powershell
# 下载并安装 WSL 2 内核更新包
# 访问: https://aka.ms/wsl2kernel

# 或使用命令
wsl --update

# 重启电脑
shutdown /r /t 0
```

---

### 错误 3: Hardware assisted virtualization must be enabled

**原因**: BIOS 中虚拟化未启用

**解决方案**: 参考上面的 "启用虚拟化" 步骤

---

### 错误 4: Docker daemon is not running

**原因**: Docker 服务未启动

**解决方案**:

```powershell
# 检查 Docker 服务
Get-Service *docker*

# 启动 Docker 服务
Start-Service docker

# 或重启 Docker Desktop
```

---

### 错误 5: version is obsolete

**警告信息**:
```
level=warning msg="docker-compose.yml: the attribute `version` is obsolete"
```

**原因**: Docker Compose V2 不再需要 version 字段

**解决方案**: 这只是警告,不影响使用。如果想去掉警告:

<function_calls>
<invoke name="read_file">
<parameter name="file_path">e:/github/metabase/docker-compose-simple.yml
