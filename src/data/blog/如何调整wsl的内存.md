---
title: "如何调整wsl的内存"
pubDatetime: 2025-06-06 09:00:00
date: 2025-06-06
tags: ["wsl"]
draft: false
description: "在 Windows Subsystem for Linux (WSL) 中，调整内存使用（包括内存上限和交换空间）需要通过 Windows 配置文件实现"
---

在 Windows Subsystem for Linux (WSL) 中，调整内存使用（包括内存上限和交换空间）需要通过 Windows 配置文件实现。以下是详细的操作步骤：

---

### **方法一：全局配置（适用于所有 WSL 发行版）**
#### **1. 创建或修改 `.wslconfig` 文件**
- 打开文件管理器，进入路径：`C:\Users\<你的用户名>\`
- 新建一个名为 `.wslconfig` 的文本文件（注意前面的点）
- 用文本编辑器（如记事本、VS Code）打开该文件

#### **2. 配置内存参数**
```ini
[wsl2]
memory=4GB        # 设置最大内存上限（根据需求调整）
swap=2GB          # 设置交换空间大小（建议为内存的一半）
processors=4      # 分配 CPU 核心数（可选）
localhostForwarding=true # 保持端口转发（可选）
```
**参数说明**：
- `memory`：WSL 虚拟机最大内存（默认无上限，建议设为物理内存的 50-70%）
- `swap`：虚拟内存大小（建议为内存值的 25-50%）
- `processors`：CPU 核心数（默认使用所有核心）

#### **3. 保存并重启 WSL**
1. 保存 `.wslconfig` 文件
2. 在 PowerShell 或 CMD 中执行：
   ```powershell
   wsl --shutdown
   ```
3. 重新启动 WSL 发行版

---

### **方法二：针对特定发行版的配置**
#### **1. 创建 `config` 文件**
- 找到发行版的安装目录（通常位于）：
  ```
  C:\Users\<用户名>\AppData\Local\Packages\<发行版文件夹>\LocalState\
  ```
- 在此目录下创建名为 `config` 的文件（无后缀名）

#### **2. 配置内存参数**
```ini
[memory]
size=4096MB  # 4GB 内存上限
[swap]
size=2048MB  # 2GB 交换空间
```

#### **3. 重启该发行版**
```powershell
wsl --terminate <发行版名称>
wsl -d <发行版名称>
```

---

### **验证内存配置是否生效**
在 WSL 终端中执行以下命令：
```bash
# 查看内存限制
cat /proc/meminfo | grep MemTotal

# 查看交换空间
free -h
```
- 输出示例：
  ```
  MemTotal: 4194304 kB  # 4GB 内存
  Swap: 2097148 kB      # 2GB 交换空间
  ```

---

### **高级配置选项（`.wslconfig` 文件）**
```ini
[wsl2]
kernel=C:\\temp\\myCustomKernel  # 自定义内核
memory=6GB                       # 6GB 内存上限
swap=3GB                         # 3GB 交换空间
swapFile=D:\\wsl-swap.vhdx       # 自定义交换文件位置
pageReporting=true               # 启用内存页报告
guiApplications=false            # 禁用 GUI 应用支持
debugConsole=true                # 启用调试控制台
```

---

### **常见问题解决**
#### **Q1：配置后 WSL 无法启动**
- 检查 `.wslconfig` 文件格式是否正确（确保无中文符号）
- 尝试删除配置并重启：
  ```powershell
  wsl --shutdown
  del %USERPROFILE%\.wslconfig
  ```

#### **Q2：内存占用仍然过高**
- 在 WSL 中限制进程内存：
  ```bash
  # 使用 cgroups 限制进程组内存
  sudo cgcreate -g memory:/wsl_limit
  sudo cgset -r memory.max=2G /wsl_limit
  cgexec -g memory:/wsl_limit your_command
  ```

#### **Q3：如何释放已用内存**
```bash
# 手动释放缓存（无需 root）
sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

---

### **推荐配置参考**
| 物理内存 | WSL 内存设置 | 交换空间设置 |
|---------|-------------|------------|
| 8GB     | 4-5GB       | 1-2GB      |
| 16GB    | 6-10GB      | 2-4GB      |
| 32GB+   | 12-16GB     | 4-8GB      |

> ⚠️ 注意：过度限制内存可能导致 WSL 运行缓慢或崩溃，建议保留至少 2GB 给 Windows 系统使用。

通过合理配置内存，可以有效平衡 WSL 和 Windows 的系统资源使用！