<div align="center">

# cac — Claude Code Cloak

**[中文](#中文) | [English](#english)**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-macOS-lightgrey.svg)]()
[![Shell](https://img.shields.io/badge/Shell-Bash-green.svg)]()

</div>

---

<a id="中文"></a>

## 中文

> **[Switch to English](#english)**

保护本地设备隐私，安全使用 Claude Code。

### 背景

Claude Code 在运行过程中会读取并上报若干设备标识符，包括硬件 UUID、安装 ID、网络出口 IP 等。

cac 通过 wrapper 机制拦截所有 `claude` 调用，在进程层面隔离设备信息，使每个使用配置对外呈现独立的设备身份。

### 隐私保护

| 保护项 | 实现方式 |
|:---|:---|
| 硬件 UUID | 拦截 `ioreg` 命令，返回配置独立的 UUID |
| 安装标识 stable_id | 切换配置时写入独立 ID |
| 用户标识 userID | 切换配置时写入独立 ID |
| 网络出口 IP | 所有流量强制走独立代理 |
| 时区 / 语言 | 根据代理出口地区自动匹配注入 |

所有 `claude` 调用（含 Agent 子进程）均通过 wrapper 拦截。代理不可达时拒绝启动，保护系统真实 IP 不被暴露。

### 安装

**一键安装（推荐）：**

```bash
curl -fsSL https://raw.githubusercontent.com/nmhjklnm/cac/master/install.sh | bash
```

安装脚本会自动完成：将 `cac` 放入 `~/bin`、在 `~/.zshrc` 中添加 PATH、生成 wrapper 和 fake ioreg。

**手动安装：**

```bash
git clone https://github.com/nmhjklnm/cac.git
cd cac
bash install.sh
```

安装完成后重开终端，或执行：

```bash
source ~/.zshrc
```

### 使用

```bash
# 添加一个使用配置（自动检测代理出口的时区和语言）
cac add us1 1.2.3.4:1080:username:password

# 切换配置（同时刷新所有隐私参数）
cac us1

# 检查当前状态
cac check

# 启动 Claude Code（走 wrapper）
claude
```

首次使用需在 Claude Code 内执行 `/login` 完成账号登录。

### 命令

| 命令 | 说明 |
|:---|:---|
| `cac add <名字> <host:port:u:p>` | 添加新配置 |
| `cac <名字>` | 切换配置，刷新所有隐私参数 |
| `cac ls` | 列出所有配置 |
| `cac check` | 检查代理连通性和当前隐私参数 |
| `cac stop` | 临时停用保护 |
| `cac -c` | 恢复保护 |

### 文件结构

```
~/.cac/
├── bin/claude          # wrapper（拦截所有 claude 调用）
├── shim-bin/ioreg      # ioreg shim，返回配置独立的硬件 UUID
├── real_claude         # 真实 claude 二进制路径
├── current             # 当前激活的配置名
├── stopped             # 存在则临时停用
└── envs/
    └── <name>/
        ├── proxy       # http://user:pass@host:port
        ├── uuid        # 独立硬件 UUID
        ├── stable_id   # 独立 stable_id
        ├── user_id     # 独立 userID
        ├── tz          # 时区（如 America/New_York）
        └── lang        # 语言（如 en_US.UTF-8）
```

### 注意事项

> **本地代理工具（Clash / Shadowrocket 等）**
> 开启 TUN 模式时，需为代理服务器 IP 添加 DIRECT 规则，保护流量不被二次拦截。

> **第三方 API 配置**
> wrapper 启动时自动清除 `ANTHROPIC_BASE_URL` / `ANTHROPIC_AUTH_TOKEN` / `ANTHROPIC_API_KEY`，确保使用官方登录端点。

> **IPv6**
> 建议在系统层关闭 IPv6，保护真实出口 IPv6 地址不被暴露。

---

<a id="english"></a>

## English

> **[切换到中文](#中文)**

Protect your local device privacy while using Claude Code.

### Background

Claude Code reads and reports several device identifiers during runtime, including hardware UUID, installation ID, network egress IP, and more.

cac intercepts all `claude` invocations via a wrapper mechanism, isolating device information at the process level so that each profile presents an independent device identity.

### Privacy Protection

| Protected Item | Implementation |
|:---|:---|
| Hardware UUID | Intercepts `ioreg` command, returns profile-specific UUID |
| Installation ID (stable_id) | Writes independent ID on profile switch |
| User ID (userID) | Writes independent ID on profile switch |
| Network Egress IP | All traffic routed through dedicated proxy |
| Timezone / Locale | Auto-detected and injected based on proxy exit region |

All `claude` invocations (including Agent subprocesses) are intercepted by the wrapper. If the proxy is unreachable, startup is blocked to prevent your real IP from being exposed.

### Installation

**One-line install (recommended):**

```bash
curl -fsSL https://raw.githubusercontent.com/nmhjklnm/cac/master/install.sh | bash
```

The install script automatically: places `cac` in `~/bin`, adds PATH to `~/.zshrc`, and generates the wrapper and fake ioreg.

**Manual install:**

```bash
git clone https://github.com/nmhjklnm/cac.git
cd cac
bash install.sh
```

After installation, restart your terminal or run:

```bash
source ~/.zshrc
```

### Usage

```bash
# Add a profile (auto-detects timezone and locale from proxy exit)
cac add us1 1.2.3.4:1080:username:password

# Switch profile (refreshes all privacy parameters)
cac us1

# Check current status
cac check

# Launch Claude Code (through wrapper)
claude
```

On first use, run `/login` inside Claude Code to authenticate.

### Commands

| Command | Description |
|:---|:---|
| `cac add <name> <host:port:u:p>` | Add a new profile |
| `cac <name>` | Switch profile, refresh all privacy parameters |
| `cac ls` | List all profiles |
| `cac check` | Check proxy connectivity and current privacy parameters |
| `cac stop` | Temporarily disable protection |
| `cac -c` | Re-enable protection |

### File Structure

```
~/.cac/
├── bin/claude          # wrapper (intercepts all claude invocations)
├── shim-bin/ioreg      # ioreg shim, returns profile-specific hardware UUID
├── real_claude         # path to the real claude binary
├── current             # currently active profile name
├── stopped             # if present, protection is temporarily disabled
└── envs/
    └── <name>/
        ├── proxy       # http://user:pass@host:port
        ├── uuid        # independent hardware UUID
        ├── stable_id   # independent stable_id
        ├── user_id     # independent userID
        ├── tz          # timezone (e.g. America/New_York)
        └── lang        # locale (e.g. en_US.UTF-8)
```

### Notes

> **Local proxy tools (Clash / Shadowrocket, etc.)**
> When TUN mode is enabled, add a DIRECT rule for the proxy server IP to prevent traffic from being double-intercepted.

> **Third-party API configuration**
> The wrapper automatically clears `ANTHROPIC_BASE_URL` / `ANTHROPIC_AUTH_TOKEN` / `ANTHROPIC_API_KEY` on startup to ensure the official login endpoint is used.

> **IPv6**
> It is recommended to disable IPv6 at the system level to prevent your real IPv6 egress address from being exposed.

---

<div align="center">

MIT License

</div>
