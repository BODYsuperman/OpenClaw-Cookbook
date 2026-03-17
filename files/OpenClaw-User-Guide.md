# OpenClaw User Guide

> Integrating the best content from the Orange Book, Datawhale tutorials, APIFox guides, and more
> 
> Version: v1.1 | Last Updated: 2026-03-17

---

## 📖 Table of Contents

1. [Understanding OpenClaw](#1-understanding-openclaw)
2. [Quick Start](#2-quick-start)
3. [Installation & Deployment](#3-installation--deployment)
4. [Channel Integration](#4-channel-integration)
5. [Configuration Guide](#5-configuration-guide)
6. [Multi-Agent Configuration](#6-multi-agent-configuration)
7. [Skills System](#7-skills-system)
8. [Memory System](#8-memory-system)
9. [Scheduled Tasks](#9-scheduled-tasks)
10. [Common Commands](#10-common-commands)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Understanding OpenClaw

### 1.1 What is OpenClaw?

OpenClaw is an **open-source, self-hosted AI Agent system** that transforms AI from a "chat tool" into a "digital employee capable of autonomously executing tasks."

| Dimension | ChatGPT | OpenClaw |
|------|---------|----------|
| Positioning | Q&A Advisor | Digital Employee |
| Deployment | Cloud SaaS | Local Self-hosted |
| Interaction | Web Chat | 20+ Chat Channels |
| Capability | Answer Questions | Execute Tasks |
| Data | Cloud Storage | Local Control |

### 1.2 Core Capabilities

- ✅ **Multi-channel Connection**: Telegram, WhatsApp, Feishu, DingTalk, Discord, etc.
- ✅ **Persistent Memory**: Remembers your preferences, projects, contacts
- ✅ **Skill Extension**: Infinitely expand capabilities through the Skills system
- ✅ **Proactive Work**: Scheduled tasks, heartbeat checks, automatic reminders
- ✅ **Local Data**: All data stored on your device
- ✅ **Multi-Agent Collaboration**: Dedicated agents for different tasks

### 1.3 Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    User Chat Channels                    │
│   Telegram / WhatsApp / Feishu / DingTalk / Discord ... │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                   Gateway Core Service                   │
│  - Message Routing  - Session Management  - Event Processing  - Task Scheduling │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                    Agent System                          │
│  main / chat / coding / Custom Agents                    │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                   Tools & Skills                         │
│  Skills / Plugins / Tools / Memory / Cron               │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Quick Start

### 2.1 System Requirements

| Requirement | Description |
|------|------|
| Node.js | v22.0.0 or higher |
| Operating System | macOS / Linux / Windows (WSL2) |
| Memory | Minimum 2GB, Recommended 4GB+ |
| Storage | Minimum 1GB available space |

### 2.2 One-Click Installation

**macOS / Linux:**
```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

**Windows (PowerShell):**
```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

### 2.3 Verify Installation

```bash
# Check version
openclaw --version

# Run diagnostics
openclaw doctor

# Check status
openclaw status
```

### 2.4 5-Minute Quick Experience

```bash
# 1. Install
npm i -g openclaw

# 2. Run setup wizard
openclaw onboard

# 3. Start Gateway
openclaw gateway start

# 4. Open dashboard
openclaw dashboard

# 5. Start chatting in your browser!
```

---

## 3. Installation & Deployment

### 3.1 Installation Methods Comparison

| Method | Difficulty | Use Case |
|------|------|----------|
| One-Click Script | ⭐ | Recommended for beginners |
| npm Manual Install | ⭐⭐ | Familiar with Node.js |
| Docker | ⭐⭐ | Containerized deployment |
| Source Build | ⭐⭐⭐⭐ | Developers |

### 3.2 Manual Installation (npm)

```bash
# Check Node.js version
node -v  # Requires >= 22.0.0

# Global installation
npm install -g openclaw@latest

# If encountering sharp module errors
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

### 3.3 Configure PATH (if needed)

If you get `openclaw: command not found`:

```bash
# Find npm global path
npm prefix -g

# Add to ~/.bashrc or ~/.zshrc
export PATH="$(npm prefix -g)/bin:$PATH"

# Apply changes
source ~/.bashrc
```

### 3.4 Docker Deployment

```bash
docker run -d \
  --name openclaw \
  -v ~/openclaw-data:/root/.openclaw \
  -p 18789:18789 \
  openclaw/openclaw:latest
```

### 3.5 Version Updates

```bash
# Update to stable version (recommended)
openclaw update --channel stable

# Update to beta version
openclaw update --channel beta

# Update to dev version
openclaw update --channel dev
```

---

## 4. Channel Integration

### 4.1 Channel Difficulty Comparison

| Tier | Platform | Time | Description |
|------|------|------|------|
| First Tier | Telegram, QQ | 5-10 min | Easiest, zero threshold |
| Second Tier | Discord, Feishu | 15-20 min | Well-documented |
| Third Tier | WhatsApp, Slack, DingTalk | 25-40 min | More configuration |
| Fourth Tier | iMessage, WeChat | 1+ hour | Additional requirements |

### 4.2 Telegram Integration (Recommended for Beginners)

**Step 1: Create Bot**
1. Search for `@BotFather` in Telegram
2. Send `/newbot` command
3. Set bot name (must end with `bot`)
4. Save the returned Bot Token

**Step 2: Configure OpenClaw**

Edit `~/.openclaw/openclaw.json`:
```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "YOUR_BOT_TOKEN",
      "dmPolicy": "pairing",
      "allowFrom": ["tg:YOUR_USER_ID"]
    }
  }
}
```

**Step 3: Start and Pair**
```bash
# Restart Gateway
openclaw gateway restart

# Send a message to the bot in Telegram
# You'll receive a pairing verification code
```

### 4.3 Feishu Integration

```bash
# Run configuration wizard
openclaw configure --section feishu
```

You need to create an enterprise self-built app in the Feishu Open Platform and obtain App ID and App Secret.

### 4.4 Multi-Channel Configuration

You can configure multiple channels simultaneously:

```json
{
  "channels": {
    "telegram": { "enabled": true, "botToken": "..." },
    "feishu": { "enabled": true, "appId": "...", "appSecret": "..." },
    "discord": { "enabled": true, "botToken": "..." }
  }
}
```

---

## 5. Configuration Guide

### 5.1 Configuration File Locations

| File | Path | Description |
|------|------|------|
| Main Config | `~/.openclaw/openclaw.json` | Gateway, channels, models |
| Agent Config | `~/.openclaw/agents/*/agent/` | Individual agent configs |
| Workspace | `~/.openclaw/workspace-*/` | Agent workspaces |

### 5.2 Model Configuration

Edit `~/.openclaw/openclaw.json`:

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "bailian": {
        "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "apiKey": "sk-YOUR_API_KEY",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen3.5-plus",
            "name": "qwen3.5-plus",
            "contextWindow": 1000000,
            "maxTokens": 65536
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "bailian/qwen3.5-plus"
      }
    }
  }
}
```

### 5.3 Recommended Models

| Provider | Model | Use Case |
|--------|------|----------|
| Alibaba Cloud | qwen3.5-plus | General tasks, cost-effective |
| Alibaba Cloud | qwen3-coder-plus | Programming, code generation |
| Zhipu | glm-5 | Optimized for Chinese tasks |
| Moonshot AI | kimi-k2.5 | Long text processing |

### 5.4 Gateway Configuration

```json
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan",
    "auth": {
      "mode": "token",
      "token": "your-secure-token"
    },
    "controlUi": {
      "allowedOrigins": ["http://192.168.1.100:18789"],
      "allowInsecureAuth": true
    }
  }
}
```

### 5.5 Authentication Modes

**Token Authentication (Recommended for API Integration):**
```json
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "your-token"
    }
  }
}
```

**Password Authentication (Recommended for Web UI):**
```json
{
  "gateway": {
    "auth": {
      "mode": "password"
    }
  }
}
```

---

## 6. Multi-Agent Configuration

### 6.1 What is Multi-Agent?

OpenClaw supports running multiple independent Agent instances, where each Agent can:
- Use different model configurations
- Have independent workspaces and memory
- Focus on specific types of tasks
- Configure Skills and tools independently

### 6.2 Built-in Agent Types

| Agent | Purpose | Recommended Model |
|-------|------|----------|
| **main** | Main conversation Agent, handles daily tasks | qwen3.5-plus |
| **chat** | Pure chat Agent, casual conversation | qwen3.5-plus |
| **coding** | Programming Agent, code generation/review | qwen3-coder-plus / glm-5 |
| **analysis** | Data analysis Agent | qwen3.5-plus |

### 6.3 Creating Custom Agents

```bash
# Create new Agent
openclaw agents create my-agent

# Set dedicated model
openclaw agents config my-agent --model glm-5

# Configure workspace
openclaw agents workspace my-agent --path ~/projects/my-agent

# Start Agent
openclaw agents start my-agent
```

### 6.4 Assign Agents to Different Tasks

| Task Type | Recommended Agent | Invocation |
|----------|-----------|----------|
| Daily conversation, Q&A | main | Default |
| Writing code, Debugging | coding | `/agent coding` |
| Data analysis, Reports | analysis | `/agent analysis` |
| Creative writing | chat | `/agent chat` |
| Domain-specific tasks | Custom Agent | `/agent <name>` |

### 6.5 Agent Routing Configuration

**Method 1: Command Prefix**
```
/agent coding Help me write a Python script
/agent analysis Analyze this data
```

**Method 2: Automatic Routing (Keyword-based)**
```json
{
  "routing": {
    "rules": [
      {
        "keywords": ["code", "programming", "python", "javascript"],
        "agent": "coding"
      },
      {
        "keywords": ["analyze", "data", "chart"],
        "agent": "analysis"
      }
    ]
  }
}
```

**Method 3: Channel Binding**
```json
{
  "channels": {
    "telegram": {
      "defaultAgent": "main"
    },
    "discord": {
      "defaultAgent": "chat"
    }
  }
}
```

### 6.6 Multi-Agent Best Practices

**✅ Recommended:**
- Create dedicated Agents for high-frequency tasks
- Use separate Agents for programming (avoid context pollution)
- Create independent Agent instances for different projects
- Regularly clean up inactive Agents

**❌ Avoid:**
- Creating too many Agents (recommended ≤5)
- All Agents using the same model (wastes resources)
- Ignoring memory isolation between Agents

### 6.7 View and Manage Agents

```bash
# List all Agents
openclaw agents list

# Check Agent status
openclaw agents status my-agent

# View Agent configuration
openclaw agents config my-agent --show

# Stop Agent
openclaw agents stop my-agent

# Delete Agent
openclaw agents delete my-agent
```

### 6.8 Multi-Agent Architecture Example

```
┌─────────────────────────────────────────────────────────┐
│                    User Message                          │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              Routing Layer (distributes by rules)        │
│   Keyword Matching / Command Prefix / Channel Binding    │
└──────────┬──────────────┬──────────────┬────────────────┘
           │              │              │
           ▼              ▼              ▼
    ┌──────────┐   ┌──────────┐   ┌──────────┐
    │   main   │   │  coding  │   │ analysis │
    │  Agent   │   │  Agent   │   │  Agent   │
    └────┬─────┘   └────┬─────┘   └────┬─────┘
         │              │              │
         ▼              ▼              ▼
    ┌──────────┐   ┌──────────┐   ┌──────────┐
    │ General  │   │  Code    │   │   Data   │
    │  Memory  │   │  Skills  │   │  Tools   │
    │ Qwen3.5  │   │  Codex   │   │  Pandas  │
    └──────────┘   └──────────┘   └──────────┘
```

---

## 7. Skills System

### 7.1 What are Skills?

Skills are OpenClaw's plugin system, enabling AI to:
- Access external APIs (weather, news, stocks)
- Operate local resources (files, cameras)
- Execute specific tasks (GitHub, calendar, email)

### 7.2 Installing Skills

```bash
# Install from ClawHub
openclaw skill install <skill-slug>

# Search Skills
openclaw skill search <keyword>

# List installed
openclaw skill list

# Update Skills
openclaw skill update
```

### 7.3 Popular Skills

| Skill | Function | Install Command |
|-------|------|----------|
| weather | Weather forecast | `skill install weather` |
| github | GitHub operations | `skill install github` |
| news-aggregator | News aggregation | `skill install news-aggregator` |
| pinchtab-browser | Browser automation | `skill install pinchtab-browser` |

### 7.4 Creating Custom Skills

```bash
# Create Skill directory
mkdir -p ~/.openclaw/workspace-main/skills/my-skill

# Create SKILL.md
cat > ~/.openclaw/workspace-main/skills/my-skill/SKILL.md << 'EOF'
# my-skill

## Description
My custom skill

## Usage
/MyCommand [arguments]
EOF
```

---

## 8. Memory System

### 8.1 Memory Types

| Type | File | Description |
|------|------|------|
| Long-term Memory | `MEMORY.md` | Curated important information |
| Daily Memory | `memory/YYYY-MM-DD.md` | Raw logs |
| Identity | `IDENTITY.md` | AI personality settings |
| User | `USER.md` | User information |
| Soul | `SOUL.md` | Behavioral guidelines |

### 8.2 Memory File Locations

```
~/.openclaw/workspace-main/
├── MEMORY.md              # Long-term memory
├── IDENTITY.md            # Identity settings
├── USER.md                # User information
├── SOUL.md                # Behavioral guidelines
├── AGENTS.md              # Workspace description
├── TOOLS.md               # Tool notes
├── HEARTBEAT.md           # Heartbeat tasks
└── memory/
    ├── 2026-03-16.md      # Daily memory
    └── 2026-03-15.md
```

### 8.3 Search Memory

```bash
# Semantic search
openclaw memory search "project mentioned last time"

# Search by date
openclaw memory search --date 2026-03-15 "meeting"
```

### 8.4 Memory Maintenance

Recommended to review MEMORY.md regularly (weekly):
- Remove outdated information
- Add new decisions and context
- Organize highlights from daily memory files

---

## 9. Scheduled Tasks

### 9.1 Cron Basics

```bash
# Add scheduled task
openclaw cron add --schedule "0 9 * * *" --message "Good morning! Today's tasks:"

# List tasks
openclaw cron list

# Run task
openclaw cron run <job-id>

# Remove task
openclaw cron remove <job-id>
```

### 9.2 Cron Expressions

| Expression | Description |
|--------|------|
| `0 9 * * *` | Daily at 9:00 |
| `0 9 * * 1` | Every Monday at 9:00 |
| `*/30 * * * *` | Every 30 minutes |
| `0 0 1 * *` | 1st of each month at 0:00 |

### 9.3 Heartbeat Tasks

Edit `HEARTBEAT.md`:

```markdown
# HEARTBEAT.md

- [ ] Check unread emails
- [ ] View calendar (within 24 hours)
- [ ] Check weather (if going out)
- [ ] Review memory files (weekly)
```

### 9.4 Practical Cron Examples

**Daily News Briefing:**
```bash
openclaw cron add \
  --name "daily-news" \
  --schedule "0 8 * * *" \
  --message "Execute news-aggregator skill, send daily news"
```

**Weekly Backup:**
```bash
openclaw cron add \
  --name "weekly-backup" \
  --schedule "0 2 * * 0" \
  --message "Backup workspace and memory directories"
```

---

## 10. Common Commands

### 10.1 Gateway Management

```bash
openclaw gateway start      # Start
openclaw gateway stop       # Stop
openclaw gateway restart    # Restart
openclaw gateway status     # Status
openclaw gateway install    # Install as system service
openclaw gateway uninstall  # Uninstall service
```

### 10.2 Configuration Management

```bash
openclaw onboard            # Initial setup wizard
openclaw configure          # Configure channels
openclaw doctor             # Health check
openclaw status             # Check status
openclaw dashboard          # Open web dashboard
openclaw tui                # Terminal interface
```

### 10.3 Model Management

```bash
openclaw models status      # Model status
openclaw models list        # List models
openclaw models add         # Add model
```

### 10.4 Channel Management

```bash
openclaw channels list      # List channels
openclaw channels add       # Add channel
openclaw channels remove    # Remove channel
```

### 10.5 Agent Management

```bash
openclaw agents list            # List Agents
openclaw agents create <name>   # Create Agent
openclaw agents config <name>   # Configure Agent
openclaw agents start <name>    # Start Agent
openclaw agents stop <name>     # Stop Agent
openclaw agents delete <name>   # Delete Agent
```

### 10.6 Memory Operations

```bash
openclaw memory search <query>   # Search memory
openclaw memory list             # List memory files
openclaw memory edit             # Edit memory
```

### 10.7 Session Management

```bash
openclaw sessions list           # List sessions
openclaw sessions history <key>  # View history
openclaw sessions send           # Send message to session
```

---

## 11. Troubleshooting

### 11.1 Common Issues

**Issue 1: `openclaw: command not found`**

```bash
# Find npm global path
npm prefix -g

# Add to PATH
export PATH="$(npm prefix -g)/bin:$PATH"

# Add to ~/.bashrc permanently
echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Issue 2: Gateway fails to start**

```bash
# Check port usage
lsof -i :18789

# View logs
journalctl --user -u openclaw-gateway.service -n 50

# Restart service
systemctl --user restart openclaw-gateway.service
```

**Issue 3: Not receiving Telegram messages**

```bash
# Check if Bot Token is correct
cat ~/.openclaw/openclaw.json | grep botToken

# Check if offset is updated
cat ~/.openclaw/telegram/update-offset-*.json

# View Gateway logs
journalctl -t gateway-start.sh -n 30 | grep telegram
```

**Issue 4: Model API call failures**

```bash
# Check API Key
openclaw models status

# Test connection
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.example.com/v1/models

# Check network
ping api.example.com
```

### 11.2 Viewing Logs

```bash
# Gateway logs
journalctl --user -u openclaw-gateway.service -f

# Last 50 entries
journalctl --user -u openclaw-gateway.service -n 50

# Specific time range
journalctl --user -u openclaw-gateway.service \
  --since "2026-03-16 10:00:00" --until "2026-03-16 12:00:00"
```

### 11.3 Diagnostic Commands

```bash
# Full diagnostics
openclaw doctor

# Check system
openclaw status

# Check configuration
openclaw config list
```

### 11.4 Reset & Recovery

```bash
# Reset configuration (use with caution)
openclaw onboard --reset

# Reinstall Gateway
openclaw gateway uninstall
openclaw gateway install

# Clear cache
rm -rf ~/.openclaw/browser
rm -rf ~/.openclaw/cron
```

---

## Appendix

### A. Resource Links

| Resource | Link |
|------|------|
| Official Docs | https://docs.openclaw.ai |
| GitHub | https://github.com/openclaw/openclaw |
| ClawHub | https://clawhub.com |
| Discord | https://discord.com/invite/clawd |

### B. Version Information

- Document Version: v1.1
- Applicable OpenClaw Version: v2026.3.11+
- Last Updated: 2026-03-17

### C. Contributors

This document integrates content from:
- OpenClaw Orange Book (Hua Shu)
- Datawhale OpenClaw Tutorial
- APIFox OpenClaw User Guide
- OpenClaw Official Documentation

---

*For questions or suggestions, please provide feedback to the OpenClaw community.*
