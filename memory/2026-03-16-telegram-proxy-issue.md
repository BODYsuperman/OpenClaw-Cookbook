# Telegram 代理问题排查记录

**日期：** 2026-03-16  
**问题：** Telegram bot 无法接收消息  
**耗时：** 约 2 小时  
**状态：** ✅ 已解决

---

## 问题现象

- Gateway 运行正常
- Telegram bot 在线
- 发送消息到 bot，Gateway 收不到
- Telegram offset 不更新
- 待处理消息积压

---

## 问题根源

### 1. 环境背景

用户 NAS 配置了系统级代理环境变量：
```bash
# ~/.bashrc 和 /etc/environment
ALL_PROXY=socks5h://127.0.0.1:1080
HTTP_PROXY=socks5h://127.0.0.1:1080
HTTPS_PROXY=socks5h://127.0.0.1:1080
NO_PROXY=localhost,127.0.0.1,192.168.0.0/16
```

### 2. 继承问题

Gateway 通过 systemd 启动时，**继承了登录会话的环境变量**，包括 `socks5h://` 代理配置。

### 3. 核心原因

**Node.js 的 `global-agent v4.1.3` 库不支持 `socks5h://` 协议！**

该库只支持：
- `http://`
- `https://`

当 Gateway 尝试通过 `socks5h://` 代理连接 Telegram API 时，`global-agent` 无法正确处理，导致连接失败。

### 4. 错误日志

```
[telegram] env proxy dispatcher init failed: Invalid URL protocol: 
the URL must start with `http:` or `https:`.

[telegram] fetch fallback: enabling sticky IPv4-only dispatcher 
(codes=UND_ERR_CONNECT_TIMEOUT)

[telegram] message failed: Network request for 'sendMessage' failed!
```

---

## 排查过程

### 尝试的失败方案

| 方案 | 结果 | 原因 |
|------|------|------|
| 修改 systemd `Environment` | ❌ | systemd 用户会话继承的变量无法覆盖 |
| 使用 `env -i` 清空环境 | ❌ | 脚本本身又设置了 socks5h 代理 |
| 使用 `UnsetEnvironment` | ❌ | systemd 从多个来源继承，无法完全清除 |
| 修改 `/etc/environment` | ❌ | 无权限 |
| `unset` 当前 shell 变量 | ❌ | 只对当前 shell 有效，不影响 systemd |

### 关键发现

1. **本机直连 Telegram 可通**（无需代理）
   ```bash
   curl -s "https://api.telegram.org/botXXX/getMe"  # 成功
   ```

2. **v2ray 提供两个代理端口**：
   - `1080` - SOCKS 代理
   - `1081` - HTTP 代理 ← 应该用这个！

3. **global-agent 支持 HTTP 代理**：
   ```bash
   HTTP_PROXY=http://127.0.0.1:1081  # ✓ 支持
   ALL_PROXY=socks5h://127.0.0.1:1080  # ✗ 不支持
   ```

---

## 最终解决方案

### 修改文件

**`/home/cc/.openclaw/scripts/gateway-start.sh`**

```bash
#!/bin/bash
# 使用 HTTP 代理 (v2ray 在 1081 端口提供 HTTP 代理)
exec env -i \
  HOME=/home/cc \
  PATH=/home/cc/.nvm/versions/node/v22.22.1/bin:/usr/local/bin:/usr/bin:/bin \
  NODE_ENV=production \
  HTTP_PROXY=http://127.0.0.1:1081 \
  HTTPS_PROXY=http://127.0.0.1:1081 \
  ALL_PROXY=http://127.0.0.1:1081 \
  GLOBAL_AGENT_HTTP_PROXY=http://127.0.0.1:1081 \
  GLOBAL_AGENT_HTTPS_PROXY=http://127.0.0.1:1081 \
  NO_PROXY=localhost,192.168.0.0/16,.local \
  no_proxy=localhost,192.168.0.0/16,.local \
  /home/cc/.nvm/versions/node/v22.22.1/bin/node \
  /home/cc/.nvm/versions/node/v22.22.1/lib/node_modules/openclaw/dist/index.js \
  gateway --port 18789
```

### 清理 systemd drop-in

```bash
rm -rf ~/.config/systemd/user/openclaw-gateway.service.d/
systemctl --user daemon-reload
```

### systemd service 配置

**`~/.config/systemd/user/openclaw-gateway.service`**

```ini
[Unit]
Description=OpenClaw Gateway (v2026.3.11)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/home/cc/.openclaw/scripts/gateway-start.sh
Restart=always
RestartSec=5
TimeoutStopSec=30
TimeoutStartSec=30
SuccessExitStatus=0 143
KillMode=control-group
Environment=HOME=/home/cc

[Install]
WantedBy=default.target
```

---

## 验证结果

```bash
# 检查进程环境变量
ps aux | grep openclaw-gateway | grep -v grep | awk '{print $2}' | \
  xargs -I{} cat /proc/{}/environ | tr '\0' '\n' | grep -i proxy

# 输出应该是 HTTP 代理，不是 socks5h://
HTTP_PROXY=http://127.0.0.1:1081
HTTPS_PROXY=http://127.0.0.1:1081
```

```bash
# 检查 Telegram offset 是否更新
cat ~/.openclaw/telegram/update-offset-coding.json
# lastUpdateId 应该持续更新
```

---

## 经验教训

### 1. Node.js global-agent 限制

- ❌ 不支持：`socks5://`, `socks5h://`
- ✅ 支持：`http://`, `https://`

### 2. 环境变量继承

- systemd 用户服务会继承登录会话的环境变量
- 使用 `env -i` 在脚本中清空环境是可靠的方法
- 避免在启动脚本中设置 `socks5h://` 代理

### 3. v2ray 配置

v2ray 通常提供两个端口：
- SOCKS 代理（默认 1080）
- HTTP 代理（默认 1081）

对于 Node.js 应用，**优先使用 HTTP 代理端口**。

### 4. 排查步骤

当遇到类似问题时：

1. **检查进程环境变量**
   ```bash
   cat /proc/<PID>/environ | tr '\0' '\n' | grep -i proxy
   ```

2. **测试代理协议支持**
   ```bash
   # 测试 HTTP 代理
   env HTTP_PROXY=http://127.0.0.1:1081 \
     curl -s "https://api.telegram.org/botXXX/getMe"
   
   # 测试直连
   curl -s "https://api.telegram.org/botXXX/getMe"
   ```

3. **检查 Gateway 日志**
   ```bash
   journalctl -t gateway-start.sh --no-pager -n 50 | \
     grep -E "(proxy|telegram|error)"
   ```

---

## 永久生效保证

- ✅ 脚本文件保存在磁盘
- ✅ systemd service 配置正确
- ✅ 不依赖登录会话环境变量
- ✅ NAS 重启后自动恢复

---

## 相关文件

| 文件 | 用途 |
|------|------|
| `/home/cc/.openclaw/scripts/gateway-start.sh` | Gateway 启动脚本（含代理配置） |
| `~/.config/systemd/user/openclaw-gateway.service` | systemd 服务配置 |
| `~/.openclaw/telegram/update-offset-*.json` | Telegram offset 状态 |

---

## 参考链接

- [global-agent npm](https://www.npmjs.com/package/global-agent)
- [v2ray 配置文档](https://www.v2ray.com/)
- [OpenClaw 文档](https://docs.openclaw.ai)
