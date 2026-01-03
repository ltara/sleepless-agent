<div align="center">

# Sleepless Agent

**一个在您睡觉时工作的 24/7 代理操作系统**

[![Documentation](https://img.shields.io/badge/Documentation-007ACC?style=for-the-badge&logo=markdown&logoColor=white)](https://context-machine-lab.github.io/sleepless-agent/)
[![DeepWiki](https://img.shields.io/badge/DeepWiki-582C83?style=for-the-badge&logo=wikipedia&logoColor=white)](https://deepwiki.com/context-machine-lab/sleepless-agent)
[![WeChat](https://img.shields.io/badge/WeChat-07C160?style=for-the-badge&logo=wechat&logoColor=white)](./assets/wechat.png)
[![Discord](https://img.shields.io/badge/Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/74my3Wkn)

</div>

拥有 Claude Code Pro 但晚上却没有使用？将它转变为一个在您睡觉时处理想法和任务的代理操作系统。这是一个基于 Claude Code CLI 和 Python Agent SDK 的 24/7 AI 助手守护进程，通过 Slack 处理随机想法和严肃任务，并提供隔离的工作空间。

## 📰 新闻

- **[2025-10-26]** 🎉 初始发布 v0.1.0 - 完整的代理操作系统，支持多代理工作流
- **[2025-10-25]** 🚀 添加了可配置策略的任务自动生成
- **[2025-10-24]** 🔧 集成了 Git 管理和自动 PR 创建
- **[2025-10-23]** 📊 实现了隔离工作空间以支持并行任务执行
- **[2025-10-22]** 💡 添加了 Claude Code Python Agent SDK 集成

## 🎬 演示

<div align="center">
  <img src="assets/cli.png" alt="Sleepless Agent CLI Demo" width="800">
  <p><em>Sleepless Agent CLI 实际操作 - 管理任务、检查状态和生成报告</em></p>
</div>

### 快速示例

```bash
# 启动守护进程
$ sle daemon
2025-10-26 03:30:12 | INFO | Sleepless Agent starting...
2025-10-26 03:30:12 | INFO | Slack bot connected

# 通过 Slack 提交任务
/think Implement OAuth2 authentication -p backend

# 检查状态
$ sle check
╭─────────────────── System Status ───────────────────╮
│ 🟢 Daemon: Running                                  │
│ 📊 Queue: 3 pending, 1 in_progress                  │
│ 💻 Usage: 45% (Day threshold: 95%)                  │
│ 🔄 Last task: "Implement OAuth2..." (in progress)   │
╰─────────────────────────────────────────────────────╯

# 查看结果
$ sle report 42
Task #42: ✅ Completed
Branch: feature/backend-42
PR: https://github.com/user/repo/pull/123
```

## ✨ 功能特性

- 🤖 **持续运行**：运行 24/7 守护进程，随时准备处理新任务
- 💬 **Slack 集成**：通过 Slack 命令提交任务
- 💭 **交互式聊天模式**：在 Slack 线程中与 Claude 进行实时对话会话
- 🎯 **混合自主性**：自动应用随机想法，严肃任务需要审查
- ⚡ **智能调度**：基于优先级优化任务执行
- 📊 **任务队列**：基于 SQLite 的持久化任务管理
- 🔌 **Claude Code SDK**：使用 Python Agent SDK 与 Claude Code CLI 交互
- 🏗️ **隔离工作空间**：每个任务都有自己的工作空间以实现真正的并行性
- 📝 **结果存储**：所有输出都保存了元数据以供将来参考

## ⚙️ 前置要求

- Python 3.11+
- 已安装 Claude Code CLI（`npm install -g @anthropic-ai/claude-code`）
- Git（用于自动提交）
- Slack 工作空间管理员访问权限（可选，仅使用 Slack 接口时需要）
- gh CLI（可选，用于 PR 自动化）

## 🚀 快速开始

### 1. 安装

```bash
pip install sleepless-agent
```

或用于开发：
```bash
git clone <repo>
cd sleepless-agent
python -m venv venv
source venv/bin/activate  # Windows 上使用 venv\Scripts\activate
pip install -e .
```

### 2. 配置 Claude Code 认证

Sleepless Agent 支持两种认证方式：

**方式 1：订阅账号（推荐用于个人开发）**
```bash
# 使用 Claude Code CLI 登录
claude login
```

**方式 2：API Key（推荐用于团队/企业）**
```bash
# 在 .env 文件中设置 API Key
ANTHROPIC_AUTH_TOKEN=sk-ant-your-api-key
```

> 💡 详细的 API Key 配置指南请参阅 [docs/api-key-setup.md](docs/api-key-setup.md)

### 3. 配置环境

```bash
cp .env.example .env
nano .env  # 根据需要编辑配置
```

**必需配置：**
- `AGENT_WORKSPACE_ROOT` - 工作空间路径（默认：./workspace）
- `AGENT_DB_PATH` - 数据库路径（默认：./workspace/data/tasks.db）

**可选配置：**
- `SLACK_BOT_TOKEN` - Slack 机器人令牌（仅使用 Slack 接口时需要）
- `SLACK_APP_TOKEN` - Slack 应用令牌（仅使用 Slack 接口时需要）
- `ANTHROPIC_AUTH_TOKEN` - API Key（使用 API Key 认证时需要）
- `LOG_LEVEL` - 日志级别（默认：INFO）

### 4. 运行

**使用命令行接口（无需 Slack）：**
```bash
# 启动守护进程
sle daemon

# 在另一个终端中提交任务
sle think "实现 OAuth2 认证" -p backend

# 检查系统状态
sle check

# 查看任务报告
sle report 42
```

**使用 Slack 接口（需要配置 Slack）：**

如果您想通过 Slack 控制 Sleepless Agent，请先完成以下设置：

#### 设置 Slack 应用

访问 https://api.slack.com/apps 并创建一个新应用：

**基本信息**
- 选择 "From scratch"
- 名称："Sleepless Agent"
- 选择您的工作空间

**启用 Socket 模式**
- 设置 > Socket Mode > 切换为 ON
- 生成应用令牌（以 `xapp-` 开头）

**创建斜杠命令**
设置 > Slash Commands > Create New Command：
- `/think` - 捕获想法或任务（使用 `-p project-name` 表示严肃任务）
- `/chat` - 启动与 Claude 的交互式聊天模式
- `/check` - 检查队列状态
- `/usage` - 显示 Claude Code Pro 计划使用情况
- `/cancel` - 取消任务或项目
- `/report` - 显示报告或任务详情
- `/trash` - 管理回收站（列出、恢复、清空）

**OAuth 范围**
Features > OAuth & Permissions > Bot Token Scopes：
- `chat:write`
- `commands`
- `app_mentions:read`
- `channels:history`（用于聊天模式）
- `groups:history`（用于私有频道的聊天模式）
- `reactions:write`（用于聊天模式指示器）

**事件订阅**（用于聊天模式）
Features > Event Subscriptions > Enable Events > Subscribe to bot events：
- `message.channels`
- `message.groups`

**安装应用**
- 安装到工作空间
- 获取机器人令牌（以 `xoxb-` 开头）
- 在 `.env` 文件中设置 `SLACK_BOT_TOKEN` 和 `SLACK_APP_TOKEN`

然后启动守护进程：

```bash
sle daemon
```

您应该看到类似的启动日志：
```
2025-10-24 23:30:12 | INFO     | sleepless_agent.core.daemon.run:178 Sleepless Agent starting...
2025-10-24 23:30:12 | INFO     | sleepless_agent.interfaces.bot.start:50 Slack bot started and listening for events (if configured)
```
日志使用 Rich 渲染以提高可读性；设置 `SLEEPLESS_LOG_LEVEL=DEBUG` 以增加详细程度。


## ⌨️ 命令行接口（CLI）使用指南

Sleepless Agent 提供了完整的 CLI 支持，无需配置 Slack 即可使用所有核心功能。

### 基本用法

安装项目后，使用 `sle` 命令：

```bash
# 启动守护进程
sle daemon

# 或在开发环境中直接运行
python -m sleepless_agent.interfaces.cli daemon
```

### 任务管理命令

| 命令 | 用途 | 示例 |
|------|------|------|
| `sle think <description>` | 捕获随机想法 | `sle think "研究 Rust 异步模式"` |
| `sle think <description> -p <project>` | 向项目添加严肃任务 | `sle think "实现 OAuth2" -p backend` |
| `sle check` | 显示系统状态和队列信息 | `sle check` |
| `sle usage` | 显示 Claude Code 使用情况 | `sle usage` |
| `sle cancel <id>` | 取消任务或项目 | `sle cancel 5` 或 `sle cancel my-app` |

### 报告和监控命令

| 命令 | 用途 | 示例 |
|------|------|------|
| `sle report` | 显示今日报告 | `sle report` |
| `sle report <task_id>` | 显示特定任务详情 | `sle report 42` |
| `sle report <date>` | 显示指定日期报告 | `sle report 2025-10-22` |
| `sle report <project>` | 显示项目报告 | `sle report backend` |
| `sle report --list` | 列出所有可用报告 | `sle report --list` |

### 回收站管理命令

| 命令 | 用途 | 示例 |
|------|------|------|
| `sle trash list` | 列出回收站中的项目 | `sle trash list` |
| `sle trash restore <project>` | 恢复已删除的项目 | `sle trash restore my-app` |
| `sle trash empty` | 清空回收站 | `sle trash empty` |

### 完整工作流示例

**1. 启动守护进程**
```bash
# 在终端 1 中启动守护进程
sle daemon
```

**2. 提交任务（在另一个终端）**
```bash
# 提交随机想法
sle think "探索 Python 新特性"

# 提交严肃任务到项目
sle think "添加用户认证功能" -p backend
sle think "修复支付接口 bug" -p payments
```

**3. 监控进度**
```bash
# 查看系统状态
sle check

# 输出示例：
# ╭─────────────────── System Status ───────────────────╮
# │ 🟢 Daemon: Running                                  │
# │ 📊 Queue: 3 pending, 1 in_progress                  │
# │ 💻 Usage: 45% (Day threshold: 20%)                  │
# │ 🔄 Last task: "添加用户认证..." (in progress)        │
# ╰─────────────────────────────────────────────────────╯
```

**4. 查看结果**
```bash
# 查看特定任务报告
sle report 42

# 输出示例：
# Task #42: ✅ Completed
# Description: 添加用户认证功能
# Project: backend
# Branch: feature/backend-42
# PR: https://github.com/user/repo/pull/123
# Files changed: 12 files
# Duration: 15 minutes
```

### 高级用法

**自定义数据库和结果路径**
```bash
sle --db-path ./custom/tasks.db --results-path ./custom/results check
```

**调试模式**
```bash
LOG_LEVEL=DEBUG sle daemon
```

**仅使用 CLI（不配置 Slack）**
```bash
# .env 文件中仅需配置：
AGENT_WORKSPACE_ROOT=./workspace
AGENT_DB_PATH=./workspace/data/tasks.db
AGENT_RESULTS_PATH=./workspace/data/results

# 启动守护进程
sle daemon

# 提交任务
sle think "实现新功能" -p my-project
```


## 💬 Slack 命令（可选）

> 💡 **注意**：Slack 集成是可选的。您可以完全使用 CLI 来控制 Sleepless Agent。

所有 Slack 命令都与 CLI 命令对齐以保持一致性：

### 📋 任务管理

| 命令 | 用途 | 示例 |
|---------|---------|---------|
| `/think` | 捕获随机想法 | `/think Explore async ideas` |
| `/think -p <project>` | 向项目添加严肃任务 | `/think Add OAuth2 support -p backend` |
| `/check` | 显示系统状态 | `/check` |
| `/usage` | 显示 Claude Code Pro 使用情况 | `/usage` |
| `/cancel` | 取消任务或项目 | `/cancel 5` 或 `/cancel my-app` |

### 💭 交互式聊天模式

在专用的 Slack 线程中开始与 Claude 的实时对话：

| 命令 | 用途 | 示例 |
|---------|---------|---------|
| `/chat <project>` | 为项目启动聊天模式 | `/chat my-backend` |
| `/chat end` | 结束当前聊天会话 | `/chat end` |
| `/chat status` | 检查活动会话状态 | `/chat status` |
| `/chat help` | 显示聊天模式帮助 | `/chat help` |

**聊天模式功能：**
- 🧵 每个会话专用线程
- 💬 维护完整的对话历史
- 🔄 实时处理指示器
- 📁 Claude 可以在项目工作空间中读取/写入/编辑文件
- ⏱️ 30 分钟不活动后自动超时
- 在线程中输入 `exit` 结束会话

> 💡 **注意**：当您运行 `/chat <project>` 时，会创建一个新线程。您的所有提示必须在**此线程内**发送 - Claude 仅响应聊天线程内的消息，而不是主频道中的消息。

### 📊 报告和回收站

| 命令 | 用途 | 示例 |
|---------|---------|---------|
| `/report` | 今日报告、任务详情、日期/项目报告或列出全部 | `/report`、`/report 42`、`/report 2025-10-22`、`/report my-app`、`/report --list` |
| `/trash` | 列出、恢复或清空回收站 | `/trash list`、`/trash restore my-app`、`/trash empty` |

> 💡 **提示**：所有 Slack 命令都有对应的 CLI 命令。详见上方的 [命令行接口（CLI）使用指南](#%EF%B8%8F-命令行接口cli使用指南) 部分。

## 🏗️ 架构

```
Slack Bot
    ↓
Slack Commands → Task Queue (SQLite)
    ↓
Agent Daemon (Event Loop)
    ↓
Claude Executor (Claude Code CLI)
    ↓
Result Manager (Storage + Git)
```

### 组件

- **daemon.py**：主事件循环、任务编排
- **bot.py**：Slack 接口、命令解析
- **task_queue.py**：任务 CRUD、优先级调度
- **claude_code_executor.py**：带隔离工作空间管理的 Python Agent SDK 包装器
- **results.py**：结果存储、文件管理
- **models.py**：Task、Result 的 SQLAlchemy 模型
- **config.yaml**：配置默认值
- **git_manager.py**：Git 自动化（提交、PR）
- **monitor.py**：健康检查和指标

## 📁 文件结构

```
sleepless-agent/
├── src/sleepless_agent/
│   ├── __init__.py
│   ├── daemon.py           # 主事件循环
│   ├── bot.py              # Slack 接口
│   ├── task_queue.py       # 任务管理
│   ├── claude_code_executor.py  # Claude CLI 包装器
│   ├── scheduler.py        # 智能调度
│   ├── git_manager.py      # Git 自动化
│   ├── monitor.py          # 健康和指标
│   ├── models.py           # 数据库模型
│   ├── results.py          # 结果存储
│   └── config.yaml         # 配置默认值
├── workspace/              # 所有持久化数据和任务工作空间
│   ├── data/               # 持久化存储
│   │   ├── tasks.db        # SQLite 数据库
│   │   ├── results/        # 任务输出文件
│   │   ├── reports/        # 每日 markdown 报告
│   │   ├── agent.log       # 应用日志
│   │   └── metrics.jsonl   # 性能指标
│   ├── tasks/              # 任务工作空间（task_1/、task_2/ 等）
│   ├── projects/           # 项目工作空间
│   └── trash/              # 软删除的项目
├── .env                    # 密钥（不跟踪）
├── pyproject.toml          # Python 包元数据和依赖
├── README.md              # 本文件
└── docs/                  # 其他文档
```

## ⚙️ 配置

运行时设置来自通过 `.env` 加载的环境变量（见 `.env.example`）。更新这些值或在 shell 中导出它们以调整代理行为。

### 使用管理

代理自动监控 Claude Code 使用情况并根据可配置的阈值智能管理任务执行。

**工作原理：**

1. **使用监控** - 每个任务通过 `claude /usage` 命令检查使用情况
2. **基于时间的阈值** - 白天和夜间操作有不同的阈值
3. **智能调度** - 达到阈值时自动暂停任务生成
4. **自动恢复** - 使用重置时任务恢复

**基于时间的配置（可在 `config.yaml` 中配置）：**
- **夜间（默认凌晨 1 点 - 上午 9 点）：** 96% 阈值 - 代理在您睡觉时积极工作
- **白天（默认上午 9 点 - 凌晨 1 点）：** 95% 阈值 - 为您的手动使用保留容量
- 通过以下配置：`claude_code.threshold_day`、`claude_code.threshold_night`
- 时间范围通过：`claude_code.night_start_hour`、`claude_code.night_end_hour`

**可见性：**
- 仪表板：在 `sle check` 中显示使用百分比
- 日志：每次使用检查都记录当前使用情况和适用阈值
- 配置：所有阈值和时间范围可在 `config.yaml` 中调整

**达到阈值时的行为：**
- ⏸️ 新任务生成在阈值处暂停
- ✅ 运行中的任务正常完成
- 📋 待处理任务在队列中等待
- ⏱️ 使用重置时自动恢复

### Git 管理

代理深度集成 Git 以实现自动版本控制和协作：

**远程仓库配置（`config.yaml`）：**
- `git.use_remote_repo`：启用/禁用远程仓库集成
- `git.remote_repo_url`：您的远程仓库 URL（例如，`git@github.com:username/repo.git`）
- `git.auto_create_repo`：如果仓库不存在则自动创建

**Git 工作流：**
- **随机想法**：自动提交到 `thought-ideas` 分支
- **严肃任务（-p 标志）**：创建功能分支（`feature/<project>-<task_id>`）并打开 PR
- **自动提交**：每次任务完成都会触发带描述性消息的提交
- **PR 创建**：严肃任务自动创建拉取请求以供审查

**重要提示：** 运行代理之前请在 `config.yaml` 中更新 `git.remote_repo_url`！

### 多代理工作流

代理采用复杂的多代理架构来处理复杂的任务：

**代理类型（`config.yaml`）：**
- **Planner Agent**：分析任务并创建执行计划（默认最多 3 轮）
- **Worker Agent**：执行计划的任务（默认最多 3 轮）
- **Evaluator Agent**：审查和验证完成的工作（默认最多 3 轮）

**配置：**
```yaml
multi_agent_workflow:
  planner:
    enabled: true
    max_turns: 3
  worker:
    enabled: true
    max_turns: 3
  evaluator:
    enabled: true
    max_turns: 3
```

每个代理可以独立启用/禁用，并配置不同的轮次限制以控制执行深度。

### 任务自动生成

代理可以自动生成任务以在空闲时保持生产力：

**生成策略（`config.yaml`）：**
- **refine_focused（45% 权重）**：专注于完成或改进现有工作
- **balanced（35% 权重）**：基于工作空间状态混合改进和新任务
- **new_friendly（20% 权重）**：优先创建创新的新项目

**任务类型：**
- **[NEW]**：在隔离工作空间中创建新任务（`workspace/tasks/<task_id>/`）
- **[REFINE:#<id>]**：改进特定的现有任务（重用任务工作空间）
- **[REFINE]**：工作空间项目的通用改进

**工作空间约束：**
- 每个任务在自己的隔离目录中执行
- 任务仅访问其工作空间和 `workspace/shared/`
- 系统目录（`workspace/data/`）受保护
- REFINE 任务重用现有工作空间以保持连续性


## 🔧 环境变量

**必需配置：**
```bash
AGENT_WORKSPACE_ROOT=./workspace
AGENT_DB_PATH=./workspace/data/tasks.db
AGENT_RESULTS_PATH=./workspace/data/results
```

**可选配置：**
```bash
# Claude Code 认证（使用 API Key 时）
ANTHROPIC_AUTH_TOKEN=sk-ant-your-api-key

# Slack Bot（仅使用 Slack 接口时）
SLACK_BOT_TOKEN=xoxb-...
SLACK_APP_TOKEN=xapp-...

# Git 配置
GIT_USER_NAME=Sleepless Agent
GIT_USER_EMAIL=agent@sleepless.local

# 日志级别
LOG_LEVEL=INFO
```

> 💡 **注意**：大多数配置通过 `config.yaml` 完成。环境变量主要用于密钥和特定于部署的设置。完整的环境变量说明请参阅 [.env.example](.env.example) 文件。

## 📝 任务类型

代理智能处理不同的任务类型：

1. **随机想法** - 自动提交到 `thought-ideas` 分支
   ```
   /think Research async patterns in Rust
   /think What's the best way to implement caching?
   ```

2. **严肃任务** - 创建功能分支和 PR，需要审查（使用 `-p` 标志）
   ```
   /think -p backend Add authentication to user service
   /think -p payments Refactor payment processing module
   ```

## 📊 监控

### CLI 命令
```bash
sle check         # 系统状态和性能统计
sle usage         # Claude Code 使用情况
sle report --list # 可用报告
```

### Slack 命令（如果配置了 Slack）
```
/check         # 系统状态和性能统计
/usage         # Claude Code 使用情况
/report --list # 可用报告
```

## 🚢 部署

### Linux（systemd）
```bash
make install-service
sudo systemctl start sleepless-agent
```

### macOS（launchd）
```bash
make install-launchd
launchctl list | grep sleepless
```

## 💡 示例工作流

### 每日头脑风暴

**使用 CLI：**
```bash
sle think "Research new Rust async libraries"
sle think "Compare Python web frameworks"
sle think "Ideas for improving API performance"
sle check
```

**使用 Slack：**
```
/think Research new Rust async libraries
/think Compare Python web frameworks
/think Ideas for improving API performance
/check
```

### 生产修复

**使用 CLI：**
```bash
sle think "Fix authentication bug in login endpoint" -p backend
sle report 42     # 获取 PR 链接
# 审查并合并 PR
```

**使用 Slack：**
```
/think Fix authentication bug in login endpoint -p backend
/report 42     # 获取 PR 链接
# 审查并合并 PR
```

### 代码审计

**使用 CLI：**
```bash
sle think "Security audit of user service" -p backend
sle think "Performance analysis of payment module" -p payments
```

**使用 Slack：**
```
/think Security audit of user service -p backend
/think Performance analysis of payment module -p payments
```

## ⚡ 性能提示

1. **使用想法填充空闲时间** - 最大化使用
2. **批量严肃任务** - 减少上下文切换
3. **监控使用** - 观察调度器日志的使用模式
4. **审查 git 历史** - 定期检查 `thought-ideas` 分支
5. **检查指标** - 运行 `sle check` 跟踪性能

## 📦 发布

- 最新稳定版：**0.1.0** – 发布在 [PyPI](https://pypi.org/project/sleepless-agent/0.1.0/)
- 使用 `pip install -U sleepless-agent` 安装或升级
- 通过 GitHub Releases 跟踪发布说明（标签 `v0.1.0` 起）

## 📚 文档

获取更详细的信息和指南：

- **[实施计划](IMPLEMENTATION_PLAN.md)** - API Key 认证、自定义配额、多平台接口支持
- **[完整文档](https://context-machine-lab.github.io/sleepless-agent/)** - 完整的文档站点
- **[DeepWiki](https://deepwiki.com/context-machine-lab/sleepless-agent)** - 交互式知识库
- **[安装指南](docs/installation.md)** - 详细的设置说明
- **[快速开始](docs/quickstart.md)** - 快速启动和运行
- **[常见问题](docs/faq.md)** - 常见问题解答
- **[故障排除](docs/troubleshooting.md)** - 常见问题和解决方案

## 🗺️ 路线图

- [ ] **高级调度** - 带基于时间和依赖调度的优先级队列
- [ ] **每日报告** - 代理工作的每日报告

## 🙏 致谢

我们深深感谢开源社区和使 Sleepless Agent 成为可能的项目：

- **[Claude Code CLI](https://github.com/anthropics/claude-code)** - 提供强大的 AI 辅助开发基础和实现无缝集成的 Python Agent SDK
- **[Slack Bolt](https://github.com/slackapi/bolt-python)** - 提供可靠的实时消息传递和命令处理，为我们的 Slack 集成提供动力
- **[SQLAlchemy](https://www.sqlalchemy.org/)** - 提供强大的数据持久化和优雅的 ORM 来管理我们的任务队列
- **[Rich](https://github.com/Textualize/rich)** - 提供美观的终端渲染，使日志和输出在视觉上更具吸引力
- **[GitPython](https://github.com/gitpython-developers/GitPython)** - 提供全面的 Git 操作，实现我们的自动化版本控制工作流

## 🤝 贡献

我们欢迎贡献！Sleepless Agent 旨在成为 24/7 AI 开发自动化的社区资源。

请参阅我们的[贡献指南](CONTRIBUTING.md)了解：
- 开发设置和环境配置
- 代码风格和测试要求
- 如何提交拉取请求
- 社区指南和行为准则

欢迎：
- 🐛 [报告错误](https://github.com/context-machine-lab/sleepless-agent/issues/new?labels=bug)
- 💡 [建议功能](https://github.com/context-machine-lab/sleepless-agent/issues/new?labels=enhancement)
- 💬 [提出问题](https://github.com/context-machine-lab/sleepless-agent/discussions)
- 🔧 [提交拉取请求](https://github.com/context-machine-lab/sleepless-agent/pulls)

## 📖 引用

如果您在研究或项目中使用 Sleepless Agent，请引用：

```bibtex
@software{sleepless_agent_2025,
  title = {Sleepless Agent: A 24/7 AgentOS for Continuous Development},
  author = {Zhimeng Guo, Hangfan Zhang, Siyuan Xu, Huaisheng Zhu, Teng Xiao, Minhao Cheng},
  year = {2025},
  publisher = {GitHub},
  journal = {GitHub repository},
  url = {https://github.com/context-machine-lab/sleepless-agent}
}
```

## 📄 许可证

根据 [MIT License](LICENSE) 发布

## 🔧 开发

于 2025-12-15 测试 Sleepless Agent 集成。
