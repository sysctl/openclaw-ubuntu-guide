# Ubuntu 虚拟机机实体机安装 OpenClaw 完整流程

本教程适用于在 Ubuntu 虚拟/实体机环境下，通过官方脚本部署 OpenClaw。

> **注意**：新版 OpenClaw（2026.3.8+）使用配置向导，**无需手动设置模型**，向导会自动识别并配置可用模型。

## 目录
- [0. 最小化安装ubuntu 24+ 桌面版 (安装桌面版)](#0-最小化安装)
- [1. 基础环境配置（免密设置）](#1-基础环境配置免密设置)
- [2. 安装基础工具与 OpenSSH](#2-安装基础工具与-openssh)
- [3. 安装 OpenClaw (官方脚本)](#3-安装-openclaw-官方脚本)
- [4. 配置向导](#4-配置向导)
- [5. 国内 API 配置](#5-国内-api-配置)
- [6. 局域网访问配置](#6-局域网访问配置)
- [7. 浏览器扩展安装](#7-浏览器扩展安装)

---
## 0. 最小化安装ubuntu 24+ 桌面版(安装桌面版)

因为需要用到一些桌面工具,安装desktop版本,后面要远程连接,建议使用静态IP地址

## 1. 基础环境配置（免密设置）

为了在后续安装脚本运行中避免频繁输入密码，我们需要为当前用户开启 `sudo` 免密权限。

### 1.1 编辑 sudoers 配置

```bash
sudo visudo
```

### 1.2 添加免密权限

在文件末尾添加以下内容（将 `spoto` 替换为你实际的用户名）：

```
spoto ALL=(ALL) NOPASSWD: ALL
```

### 1.3 保存并退出

- `nano` 编辑器：按 `Ctrl+O` 保存，`Enter` 确认，`Ctrl+X` 退出
- `vim` 编辑器：输入 `:wq` 保存退出

---

## 2. 安装基础工具与 OpenSSH

确保系统更新，并安装远程连接及脚本下载所需的组件。

```bash
# 更新系统包列表
sudo apt update

# 安装基础下载工具
sudo apt install -y curl

# 安装并启动 OpenSSH 服务（方便通过 PC 远程管理）
sudo apt install -y openssh-server
```

### 2.1 验证 SSH 服务状态

```bash
# 检查 SSH 服务状态
sudo systemctl status ssh

# 如果服务未启动，使用以下命令启动
sudo systemctl start ssh
```

### 2.2 获取虚拟机 IP 地址

```bash
# 获取局域网 IP 地址，用于远程连接
ip addr show | grep -E "inet " | awk '{print $2}' | cut -d'/' -f1 | grep -v "^127"
```

记录此 IP 地址，后续可通过 SSH 远程管理虚拟机。

---

## 3. 安装 OpenClaw (官方脚本)

使用官方提供的一键安装脚本进行部署：

```bash
curl -fsSL https://molt.bot/install.sh | bash
```

---

## 4. 配置向导

运行配置向导完成基本设置（推荐）：

```bash
openclaw onboard --install-daemon
```

### 4.1 向导配置选项

按照向导提示完成配置：

1. **选择安全选项** - 理解风险
2. **配置工作区目录** - 设置工作文件存放位置
3. **配置国内 API**（可选）- 选择 MiniMax 或智谱，输入 API Key
4. **配置局域网访问**（可选）- 开启 LAN Access
5. **设置认证方式** - Token 认证

### 4.2 验证安装

```bash
# 检查 openclaw 是否安装成功
which openclaw

# 查看版本
openclaw --version

# 检查 Gateway 状态
openclaw gateway status
```

---

## 5. 国内 API 配置

> **与旧版教程的区别**：新版 OpenClaw 向导会自动识别并配置模型，**无需手动编辑配置文件**。

### 5.1 使用向导配置（推荐）

```bash
openclaw onboard
```

选择对应 API 服务商，输入 API Key 即可。

### 5.2 API Key 获取

| 服务商 | 地址 |
|--------|------|
| MiniMax | https://www.minimaxi.com → 控制台 → API Keys |
| 智谱 AI | https://bigmodel.cn → 控制台 → API Keys |

### 5.3 验证模型

```bash
openclaw models list
```

---

## 6. 局域网访问配置

> **注意**：配置向导可以开启局域网访问，但 **无法直接配置 controlUi**，需要手动添加。

### 6.1 使用向导开启局域网

```bash
openclaw onboard
```

选择 `Enable LAN Access` 或类似选项。

### 6.2 手动添加 controlUi 配置

向导设置后，需手动编辑配置文件：

```bash
nano ~/.openclaw/openclaw.json
```

在 gateway 配置中添加：

```json
"controlUi": {
  "dangerouslyAllowHostHeaderOriginFallback": true,
  "allowInsecureAuth": true,
  "dangerouslyDisableDeviceAuth": true
}
```

### 6.3 重启 Gateway

```bash
openclaw gateway restart
```

### 6.4 获取访问信息

```bash
# 获取局域网 IP
ip addr show | grep -E "inet " | awk '{print $2}' | cut -d'/' -f1 | grep -v "^127"

# 获取 Token
grep '"token"' ~/.openclaw/openclaw.json
```

访问地址：`http://<你的局域网IP>:18789/`

---

## 7. 浏览器扩展安装

如需在 OpenClaw 中调用浏览器功能，请在终端执行扩展安装命令：

```bash
openclaw browser extension install
```

### 7.1 验证浏览器扩展

```bash
# 检查浏览器扩展状态
openclaw browser status
```

---

## 常见问题

### Q1: sudo 免密设置失败

**问题**：执行 `sudo visudo` 时权限不足

**解决方案**：

```bash
# 切换到 root 用户
su -

# 然后执行 visudo
visudo
```

### Q2: SSH 无法连接

**问题**：从主机无法 SSH 连接虚拟机

**解决方案**：

1. 检查虚拟机防火墙：
   ```bash
   sudo ufw status
   sudo ufw allow ssh
   ```

2. 检查 SSH 服务：
   ```bash
   sudo systemctl restart ssh
   ```

3. 确认虚拟机 IP 地址正确

### Q3: OpenClaw 安装失败

**问题**：官方脚本安装失败

**解决方案**：

1. 检查系统要求：
   ```bash
   node --version  # 需要 Node.js ≥ 22
   ```

2. 清理后重试：
   ```bash
   npm uninstall -g openclaw
   npm cache clean --force
   curl -fsSL https://molt.bot/install.sh | bash
   ```

3. 查看错误日志：
   ```bash
   tail -f ~/.openclaw/logs/gateway.log
   ```

### Q4: 向导找不到

**问题**：`openclaw onboard` 命令不存在

**解决方案**：

```bash
openclaw --version
# 确保版本 ≥ 2026.1.24

# 如需更新
openclaw update
```

### Q5: 局域网无法访问

**问题**：其他设备无法访问 Web UI

**解决方案**：

1. 确认已按 6.2 节添加 controlUi 配置
2. 检查防火墙：`sudo ufw allow 18789/tcp`
3. 重启 Gateway：`openclaw gateway restart`

---

## 后续步骤

完成本教程后，你将拥有：

- ✅ 运行在 Ubuntu 虚拟机上的 OpenClaw
- ✅ 配置完成的国内 API（MiniMax 或智谱）
- ✅ 可通过局域网访问的 Web UI
- ✅ 可选的浏览器扩展功能

## 相关资源

- [OpenClaw 官方文档](https://docs.clawd.bot)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [MiniMax 官网](https://www.minimaxi.com)
- [智谱 AI 官网](https://bigmodel.cn)

---

## 许可证

MIT License - 详见 [LICENSE](./LICENSE) 文件

---

## 版本

- 更新日期: 2026-03-14
- 适用系统: Ubuntu 20.04/22.04
- 支持的 OpenClaw 版本: 2026.3.12+
