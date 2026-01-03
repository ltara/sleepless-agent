# Sleepless Agent 项目结构分析

> 这是一个详细的技术文档，深入解析 Sleepless Agent 的项目架构、核心模块和实现原理。

## 系统指令
始终使用简体中文回复

## 目录

- [项目概览](#项目概览)
- [详细目录结构](#详细目录结构)
- [核心模块详解](#核心模块详解)
- [数据流和架构](#数据流和架构)
- [配置系统详解](#配置系统详解)
- [关键功能实现原理](#关键功能实现原理)
- [开发指南](#开发指南)
- [技术亮点和创新点](#技术亮点和创新点)

---

## 项目概览

### 项目简介

**Sleepless Agent** 是一个基于 Claude Code CLI 和 Python Agent SDK 的 24/7 AI 代理操作系统。它通过 Slack 接口接收和处理任务，利用隔离工作空间实现真正的并行任务执行，并通过智能调度算法优化 Claude Code Pro 计划的使用配额。

**核心价值主张：**
- 将 Claude Code Pro 转变为不间断工作的 AI 助手
- 在您睡觉时自动处理想法和任务
- 智能区分"随机想法"和"严肃任务"，采用不同的处理策略
- 完整的 Git 集成，自动创建分支和拉取请求

### 技术栈总结

**核心技术：**
- **Python 3.11+** - 主要开发语言
- **Claude Agent SDK** - 与 Claude Code CLI 的 Python 接口
- **SQLAlchemy** - ORM 和数据库管理
- **Slack Bolt SDK** - Slack 集成和实时通信
- **GitPython** - Git 操作自动化
- **Rich** - 终端美化输出

**依赖服务：**
- **Claude Code CLI** - AI 代码生成和任务执行引擎
- **SQLite** - 任务队列和元数据存储
- **Slack** - 用户交互界面
- **Git/GitHub** - 版本控制和协作

### 核心特性概要

| 特性 | 说明 | 技术实现 |
|------|------|----------|
| **24/7 守护进程** | 持续运行，随时响应任务 | 基于事件循环的 `SleeplessAgent` 类 |
| **Slack 集成** | 通过斜杠命令提交和管理任务 | Slack Bolt SDK + Socket Mode |
| **交互式聊天模式** | 实时对话，支持多轮交互 | 线程化会话管理 + Claude SDK |
| **隔离工作空间** | 每个任务独立目录，避免冲突 | 动态工作空间创建和管理 |
| **智能调度** | 基于时间和使用配额的任务调度 | `SmartScheduler` + `BudgetManager` |
| **任务队列** | 持久化任务管理，支持优先级 | SQLite + SQLAlchemy ORM |
| **Git 自动化** | 自动提交、分支、PR 创建 | GitPython + GitHub CLI |
| **多代理工作流** | Planner/Worker/Evaluator 三阶段 | 多轮对话 + 角色提示工程 |
| **自动任务生成** | 空闲时自动创建和改进任务 | 三种可配置策略 |

---

## 详细目录结构

```
sleepless-agent/
│
├── src/sleepless_agent/                    # 核心源代码根目录
│   ├── __init__.py                         # 包初始化，导出公共 API
│   ├── __main__.py                         # CLI 入口点（sle 命令）
│   ├── config.yaml                         # 默认配置文件
│   │
│   ├── chat/                               # 💬 交互式聊天模式
│   │   ├── __init__.py
│   │   ├── executor.py                     # 聊天专用的 Claude 执行器
│   │   ├── handler.py                      # 聊天命令和消息处理
│   │   └── session.py                      # 会话状态管理（超时、历史）
│   │
│   ├── core/                               # 🧠 核心业务逻辑
│   │   ├── __init__.py
│   │   ├── daemon.py                       # 主守护进程（事件循环）
│   │   ├── executor.py                     # Claude Code SDK 执行器
│   │   ├── models.py                       # 数据模型（Task, Result）
│   │   ├── queue.py                        # 任务队列管理（CRUD）
│   │   ├── task_runtime.py                 # 任务运行时环境
│   │   └── timeout_manager.py              # 超时控制和进程管理
│   │
│   ├── interfaces/                         # 🖥️ 用户交互接口
│   │   ├── __init__.py
│   │   ├── cli.py                          # 命令行接口（sle 命令）
│   │   └── bot.py                          # Slack Bot（斜杠命令）
│   │
│   ├── storage/                            # 💾 数据存储和版本控制
│   │   ├── __init__.py
│   │   ├── git.py                          # Git 管理（分支、提交、PR）
│   │   ├── sqlite.py                       # SQLite 数据库基础类
│   │   ├── db_helpers.py                   # 数据库辅助函数
│   │   ├── results.py                      # 结果存储和检索
│   │   └── workspace.py                    # 工作空间设置和隔离
│   │
│   ├── scheduling/                         # 📅 任务调度和自动生成
│   │   ├── __init__.py
│   │   ├── scheduler.py                    # 智能调度器（使用配额管理）
│   │   ├── auto_generator.py               # 自动任务生成（三种策略）
│   │   └── time_utils.py                   # 时间工具（白天/夜间判断）
│   │
│   ├── monitoring/                         # 📊 监控和性能追踪
│   │   ├── __init__.py
│   │   ├── logging.py                      # 日志配置和管理
│   │   ├── monitor.py                      # 系统健康检查
│   │   ├── pro_plan_usage.py               # Claude Pro 使用情况追踪
│   │   └── report_generator.py             # 报告生成（日报、项目报告）
│   │
│   ├── tasks/                              # 📋 任务相关工具
│   │   ├── __init__.py
│   │   ├── refinement.py                   # 任务优化和改进逻辑
│   │   └── utils.py                        # 任务工具函数
│   │
│   ├── utils/                              # 🛠️ 通用工具函数
│   │   ├── __init__.py
│   │   ├── config.py                       # 配置加载和管理
│   │   ├── directory_manager.py            # 目录管理和清理
│   │   ├── display.py                      # 终端显示和格式化
│   │   ├── exceptions.py                   # 自定义异常类
│   │   ├── live_status.py                  # 实时状态追踪
│   │   ├── metrics_aggregator.py           # 性能指标聚合
│   │   └── readme_manager.py               # README 自动生成
│   │
│   └── deployment/                         # 🚀 部署配置
│       ├── sleepless-agent.service         # systemd 服务配置（Linux）
│       └── com.sleepless-agent.plist       # launchd 配置（macOS）
│
├── workspace/                              # 💼 工作空间（运行时数据）
│   ├── data/                               # 持久化数据
│   │   ├── tasks.db                        # SQLite 数据库
│   │   ├── results/                        # 任务结果 JSON 文件
│   │   ├── reports/                        # 每日 Markdown 报告
│   │   ├── agent.log                       # 应用日志
│   │   ├── metrics.jsonl                   # 性能指标（JSONL 格式）
│   │   └── chat_sessions.json              # 聊天会话状态
│   ├── tasks/                              # 任务工作空间（task_1/, task_2/...）
│   ├── projects/                           # 项目工作空间（严肃任务）
│   ├── shared/                             # 跨任务共享文件
│   └── trash/                              # 软删除的项目
│
├── docs/                                   # 📖 文档
│   ├── index.md                            # 文档主页
│   ├── quickstart.md                       # 快速开始
│   ├── installation.md                     # 安装指南
│   ├── troubleshooting.md                  # 故障排除
│   ├── faq.md                              # 常见问题
│   ├── changelog.md                        # 更新日志
│   │
│   ├── concepts/                           # 概念文档
│   │   ├── architecture.md                 # 架构说明
│   │   ├── task-lifecycle.md               # 任务生命周期
│   │   ├── workspace-isolation.md          # 工作空间隔离
│   │   ├── pro-plan-management.md          # Pro 计划管理
│   │   └── scheduling.md                   # 调度机制
│   │
│   ├── guides/                             # 使用指南
│   │   ├── slack-setup.md                  # Slack 设置
│   │   ├── environment-setup.md            # 环境配置
│   │   └── git-integration.md              # Git 集成
│   │
│   └── reference/                          # 参考文档
│       └── api/cli-commands.md             # CLI 命令参考
│
├── assets/                                 # 静态资源（图片、图标）
├── .github/                                # GitHub 配置（工作流、模板）
├── .env.example                            # 环境变量示例
├── pyproject.toml                          # Python 项目配置
├── Makefile                                # 开发和部署命令
├── mkdocs.yml                              # 文档配置
├── README.md                               # 项目说明（用户指南）
├── CLAUDE.md                               # 项目结构分析（本文件）
├── CONTRIBUTING.md                         # 贡献指南
├── LICENSE                                 # MIT 许可证
└── .gitignore                              # Git 忽略规则
```

---

## 核心模块详解

### 1. `core/daemon.py` - 主事件循环和控制器

**核心类：`SleeplessAgent`**

这是整个系统的心脏，负责持续运行并协调所有组件。

**主要职责：**
- 初始化所有子系统（Slack Bot、任务队列、调度器）
- 运行主事件循环（无限循环，定期检查任务）
- 协调任务执行和调度
- 生成日报和指标报告

**关键方法：**

```python
class SleeplessAgent:
    def __init__(self, config: dict):
        """初始化代理和所有子系统"""
        self.config = config
        self.task_queue = TaskQueue(db_path)
        self.scheduler = SmartScheduler(config)
        self.executor = ClaudeCodeExecutor(config)
        self.git_manager = GitManager(config)

    async def run(self):
        """主事件循环"""
        while True:
            # 1. 检查使用配额
            if not self.scheduler.should_generate_tasks():
                await asyncio.sleep(60)
                continue

            # 2. 获取下一个待处理任务
            task = self.task_queue.get_next_task()

            # 3. 执行任务
            if task:
                result = await self.executor.execute(task)
                self.task_queue.update_task(task.id, status='completed')

            # 4. 自动生成新任务（如果配置启用）
            if self.config['auto_generation']['enabled']:
                await self.auto_generate_task()

            # 5. 生成日报（每天凌晨）
            if self._is_report_time():
                await self._generate_daily_report()
```

**特点：**
- 使用 `asyncio` 实现异步执行
- 内置错误恢复机制（任务失败时重试）
- 支持优雅关闭（SIGTERM/SIGINT 处理）

---

### 2. `core/executor.py` - Claude Code SDK 执行器

**核心类：`ClaudeCodeExecutor`**

这是与 Claude Code CLI 交互的关键模块，使用 Python Agent SDK 进行通信。

**主要职责：**
- 设置隔离工作空间
- 调用 Claude Code SDK 执行任务
- 处理工具使用（Tool Use）和多轮对话
- 捕获和解析输出
- 管理超时和错误

**关键特性：**

```python
class ClaudeCodeExecutor:
    def __init__(self, config: dict):
        self.model = config['claude_code']['model']  # claude-sonnet-4-5-20250929
        self.timeout = config['agent']['task_timeout_seconds']

    async def execute(self, task: Task) -> Result:
        """执行单个任务"""
        # 1. 设置工作空间
        workspace = self._setup_workspace(task)

        # 2. 构建提示
        prompt = self._build_prompt(task)

        # 3. 调用 Claude Agent SDK
        try:
            response = await self._query_claude(
                prompt=prompt,
                workspace=workspace,
                max_turns=self.config['multi_agent_workflow']['worker']['max_turns']
            )

            # 4. 解析响应
            result = self._parse_response(response)

            # 5. 保存结果
            self._save_result(task, result)

            return result
        except TimeoutError:
            return Result(status='failed', error='Task timeout')
```

**工作空间隔离机制：**
- 每个任务在 `workspace/tasks/<task_id>/` 或 `workspace/projects/<project_name>/` 中执行
- 任务只能访问自己的工作空间和 `workspace/shared/`
- 系统目录（`workspace/data/`）受保护

---

### 3. `core/queue.py` - 任务队列管理

**核心类：`TaskQueue`**

基于 SQLite 的持久化任务队列，支持优先级调度和 CRUD 操作。

**数据模型（见 `core/models.py`）：**

```python
class Task(Base):
    __tablename__ = 'tasks'

    id = Column(Integer, primary_key=True)
    description = Column(String, nullable=False)
    priority = Column(Enum(TaskPriority))  # THOUGHT, SERIOUS, GENERATED
    status = Column(Enum(TaskStatus))      # PENDING, IN_PROGRESS, COMPLETED, FAILED
    task_type = Column(Enum(TaskType))     # NEW, REFINE
    project_name = Column(String, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    started_at = Column(DateTime, nullable=True)
    completed_at = Column(DateTime, nullable=True)
```

**关键方法：**

```python
class TaskQueue:
    def add_task(self, description: str, priority: TaskPriority,
                 project_name: str = None) -> Task:
        """添加新任务到队列"""

    def get_next_task(self) -> Optional[Task]:
        """获取下一个待处理任务（基于优先级）"""
        # 优先级顺序：SERIOUS > THOUGHT > GENERATED

    def update_task_status(self, task_id: int, status: TaskStatus):
        """更新任务状态"""

    def get_tasks_by_project(self, project_name: str) -> List[Task]:
        """获取项目的所有任务"""
```

**优先级策略：**
1. **SERIOUS（严肃任务）**：最高优先级，带 `-p` 标志的任务
2. **THOUGHT（随机想法）**：中等优先级，快速处理
3. **GENERATED（自动生成）**：最低优先级，填充空闲时间

---

### 4. `interfaces/bot.py` - Slack 集成

**核心类：`SlackBot`**

使用 Slack Bolt SDK 处理斜杠命令和消息事件。

**主要特性：**
- Socket Mode 连接（无需公网 URL）
- 斜杠命令处理（`/think`, `/check`, `/usage` 等）
- 聊天模式集成（线程消息监听）
- 实时状态反馈（emoji 反应、状态更新）

**命令处理示例：**

```python
from slack_bolt import App
from slack_bolt.adapter.socket_mode import SocketModeHandler

app = App(token=os.environ["SLACK_BOT_TOKEN"])

@app.command("/think")
def handle_think(ack, command, client):
    """处理 /think 命令"""
    ack()  # 立即确认命令

    text = command['text']

    # 解析参数
    if '-p' in text:
        # 严肃任务
        parts = text.split('-p')
        description = parts[0].strip()
        project = parts[1].strip()
        priority = TaskPriority.SERIOUS
    else:
        # 随机想法
        description = text
        project = None
        priority = TaskPriority.THOUGHT

    # 添加到队列
    task = task_queue.add_task(description, priority, project)

    # 回复用户
    client.chat_postMessage(
        channel=command['channel_id'],
        text=f"✅ 任务已添加到队列 (ID: {task.id})"
    )

@app.command("/check")
def handle_check(ack, command, client):
    """处理 /check 命令"""
    ack()

    # 获取系统状态
    status = daemon.get_status()

    # 格式化输出
    message = f"""
╭─────────────────── System Status ───────────────────╮
│ 🟢 Daemon: {status['daemon_status']}
│ 📊 Queue: {status['pending_tasks']} pending, {status['in_progress_tasks']} in_progress
│ 💻 Usage: {status['usage_percent']}% (Threshold: {status['threshold']}%)
│ 🔄 Last task: "{status['last_task']}" ({status['last_task_status']})
╰─────────────────────────────────────────────────────╯
    """

    client.chat_postMessage(channel=command['channel_id'], text=message)
```

---

### 5. `storage/git.py` - Git 自动化

**核心类：`GitManager`**

使用 GitPython 实现 Git 操作自动化。

**主要功能：**
- 自动创建分支（`thought-ideas` 或 `feature/<project>-<task_id>`）
- 自动提交（带描述性消息）
- 自动推送到远程仓库
- 自动创建 Pull Request（使用 `gh` CLI）

**工作流示例：**

```python
class GitManager:
    def commit_task_result(self, task: Task, result: Result):
        """提交任务结果到 Git"""

        if task.priority == TaskPriority.THOUGHT:
            # 随机想法 -> thought-ideas 分支
            branch = 'thought-ideas'
            self._ensure_branch(branch)
            self._commit_changes(
                message=f"💡 Thought: {task.description}",
                branch=branch
            )
        elif task.priority == TaskPriority.SERIOUS:
            # 严肃任务 -> 功能分支 + PR
            branch = f"feature/{task.project_name}-{task.id}"
            self._create_branch(branch)
            self._commit_changes(
                message=f"✨ {task.description}",
                branch=branch
            )
            self._create_pull_request(
                branch=branch,
                title=task.description,
                project=task.project_name
            )

    def _create_pull_request(self, branch: str, title: str, project: str):
        """使用 gh CLI 创建 PR"""
        subprocess.run([
            'gh', 'pr', 'create',
            '--title', title,
            '--body', f"Project: {project}\n\nAuto-generated by Sleepless Agent",
            '--head', branch,
            '--base', 'main'
        ])
```

---

### 6. `scheduling/scheduler.py` - 智能调度

**核心类：`SmartScheduler` 和 `BudgetManager`**

基于时间和使用配额的智能调度系统。

**关键特性：**
- 时间感知（白天 vs 夜间）
- 使用配额监控（通过 `claude /usage` 命令）
- 动态阈值调整
- 任务生成控制

**配额管理逻辑：**

```python
class BudgetManager:
    def __init__(self, config: dict):
        self.threshold_day = config['claude_code']['threshold_day']      # 95%
        self.threshold_night = config['claude_code']['threshold_night']  # 96%
        self.night_start = config['claude_code']['night_start_hour']    # 1
        self.night_end = config['claude_code']['night_end_hour']        # 9

    def get_current_usage(self) -> float:
        """获取当前使用百分比"""
        result = subprocess.run(['claude', '/usage'], capture_output=True, text=True)
        # 解析输出，提取使用百分比
        return self._parse_usage(result.stdout)

    def should_allow_task_generation(self) -> bool:
        """判断是否允许生成新任务"""
        usage = self.get_current_usage()
        threshold = self._get_current_threshold()

        return usage < threshold

    def _get_current_threshold(self) -> float:
        """根据时间返回适当的阈值"""
        current_hour = datetime.now().hour
        if self.night_start <= current_hour < self.night_end:
            return self.threshold_night  # 夜间更激进
        else:
            return self.threshold_day     # 白天保守
```

---

### 7. `chat/handler.py` - 交互式聊天模式

**核心类：`ChatHandler`**

管理 Slack 线程中的实时对话。

**关键特性：**
- 会话隔离（每个线程一个会话）
- 对话历史维护
- 30 分钟不活动超时
- 实时处理指示器（emoji 反应）

**会话管理：**

```python
class ChatSession:
    def __init__(self, project: str, thread_ts: str):
        self.project = project
        self.thread_ts = thread_ts  # Slack 线程 ID
        self.history = []           # 对话历史
        self.last_activity = datetime.now()

    def add_message(self, role: str, content: str):
        """添加消息到历史"""
        self.history.append({"role": role, "content": content})
        self.last_activity = datetime.now()

    def is_expired(self) -> bool:
        """检查会话是否超时"""
        return (datetime.now() - self.last_activity).seconds > 1800  # 30 分钟

class ChatHandler:
    def handle_message(self, message: dict, client):
        """处理聊天消息"""
        thread_ts = message['thread_ts']
        session = self.sessions.get(thread_ts)

        if not session or session.is_expired():
            client.chat_postMessage(
                channel=message['channel'],
                thread_ts=thread_ts,
                text="❌ 会话已过期，请使用 /chat <project> 重新开始"
            )
            return

        # 添加用户消息到历史
        session.add_message("user", message['text'])

        # 调用 Claude（带历史）
        response = self.executor.chat(
            prompt=message['text'],
            history=session.history,
            workspace=f"workspace/projects/{session.project}"
        )

        # 添加助手响应到历史
        session.add_message("assistant", response)

        # 回复到线程
        client.chat_postMessage(
            channel=message['channel'],
            thread_ts=thread_ts,
            text=response
        )
```

---

## 数据流和架构

### 完整架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                          用户界面层                               │
├──────────────────┬──────────────────────────────────────────────┤
│  Slack Bot       │  CLI (sle)                                    │
│  - /think        │  - think <desc> -p <project>                  │
│  - /check        │  - check                                      │
│  - /chat         │  - report [id]                                │
│  - /usage        │  - cancel <id>                                │
│  - /report       │  - trash [subcmd]                             │
└────────┬─────────┴───────────────────┬──────────────────────────┘
         │                             │
         v                             v
┌─────────────────────────────────────────────────────────────────┐
│                       核心控制层                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │  SleeplessAgent (daemon.py)                            │     │
│  │  - 主事件循环                                           │     │
│  │  - 任务协调                                             │     │
│  │  - 子系统管理                                           │     │
│  └────────────────────────────────────────────────────────┘     │
└───────┬─────────────────┬─────────────────┬────────────────────┘
        │                 │                 │
        v                 v                 v
┌───────────────┐  ┌──────────────┐  ┌─────────────────────┐
│  TaskQueue    │  │  Scheduler   │  │  ClaudeExecutor     │
│  (queue.py)   │  │ (scheduler.py)│  │  (executor.py)      │
│               │  │               │  │                     │
│  - CRUD       │  │  - 使用监控  │  │  - SDK 调用         │
│  - 优先级     │  │  - 阈值管理  │  │  - 工作空间隔离     │
│  - 持久化     │  │  - 时间感知  │  │  - 多轮对话         │
└───────┬───────┘  └──────┬───────┘  └─────────┬───────────┘
        │                 │                     │
        v                 v                     v
┌─────────────────────────────────────────────────────────────────┐
│                       存储层                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐    │
│  │  SQLite DB   │  │  File System │  │  Git Repository    │    │
│  │  (tasks.db)  │  │  (results/)  │  │  (remote)          │    │
│  │              │  │              │  │                    │    │
│  │  - Tasks     │  │  - Results   │  │  - Branches        │    │
│  │  - Results   │  │  - Reports   │  │  - Commits         │    │
│  │  - Metadata  │  │  - Logs      │  │  - Pull Requests   │    │
│  └──────────────┘  └──────────────┘  └────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### 任务生命周期流程图

```
1. 任务创建 (Task Creation)
   ↓
   ├─ 来源 A: Slack 命令 (/think)
   ├─ 来源 B: CLI 命令 (sle think)
   └─ 来源 C: 自动生成 (AutoGenerator)
   ↓
2. 入队 (Enqueue)
   ↓
   TaskQueue.add_task()
   ↓
   写入 SQLite (status=PENDING)
   ↓
3. 调度检查 (Scheduling Check)
   ↓
   SmartScheduler.should_generate_tasks()
   ↓
   ├─ 检查使用配额 (claude /usage)
   ├─ 检查时间窗口 (白天/夜间)
   └─ 检查阈值 (95% or 96%)
   ↓
   [如果超过阈值] → 暂停，等待重置
   [如果未超过] → 继续
   ↓
4. 任务选择 (Task Selection)
   ↓
   TaskQueue.get_next_task()
   ↓
   优先级排序：SERIOUS > THOUGHT > GENERATED
   ↓
5. 工作空间设置 (Workspace Setup)
   ↓
   ├─ 创建隔离目录 (workspace/tasks/<id>/ 或 projects/<name>/)
   ├─ 设置权限和边界
   └─ 初始化 Git（如果需要）
   ↓
6. 执行 (Execution)
   ↓
   ClaudeCodeExecutor.execute(task)
   ↓
   ├─ [多代理工作流启用?]
   │   ├─ Phase 1: Planner Agent (分析任务)
   │   ├─ Phase 2: Worker Agent (执行任务)
   │   └─ Phase 3: Evaluator Agent (验证结果)
   │
   └─ [直接执行模式]
       └─ 单轮或多轮对话
   ↓
7. 结果处理 (Result Processing)
   ↓
   ├─ 保存结果 JSON (workspace/data/results/<id>.json)
   ├─ 更新任务状态 (status=COMPLETED or FAILED)
   ├─ 记录指标 (metrics.jsonl)
   └─ 生成报告（如果需要）
   ↓
8. Git 集成 (Git Integration)
   ↓
   GitManager.commit_task_result()
   ↓
   ├─ [THOUGHT 任务]
   │   ├─ 切换到 thought-ideas 分支
   │   ├─ 提交更改
   │   └─ 推送到远程
   │
   └─ [SERIOUS 任务]
       ├─ 创建功能分支 (feature/<project>-<id>)
       ├─ 提交更改
       ├─ 推送到远程
       └─ 创建 Pull Request (gh pr create)
   ↓
9. 通知 (Notification)
   ↓
   ├─ Slack 消息（任务完成通知）
   └─ 日志记录（agent.log）
   ↓
10. 清理 (Cleanup)
    ↓
    ├─ 可选：归档工作空间
    └─ 更新统计数据
```

### 数据流向说明

**用户交互 → 任务队列：**
1. 用户通过 Slack 或 CLI 提交任务
2. 命令解析器提取描述、优先级、项目名
3. 创建 Task 对象并写入 SQLite

**任务队列 → 执行器：**
1. 守护进程从队列获取下一个任务（基于优先级）
2. 调度器检查是否允许执行（使用配额）
3. 如果允许，任务传递给执行器

**执行器 → Claude Code SDK：**
1. 执行器设置工作空间环境
2. 构建提示（包含任务描述、上下文）
3. 调用 Claude Agent SDK
4. 处理工具使用和多轮对话
5. 解析最终响应

**执行器 → 存储层：**
1. 结果保存为 JSON 文件
2. 任务状态更新到数据库
3. 指标写入 metrics.jsonl
4. 如果启用，更改提交到 Git

---

## 配置系统详解

### `config.yaml` 完整配置项

```yaml
# Claude Code 配置
claude_code:
  binary_path: claude                        # Claude CLI 可执行文件路径
  model: claude-sonnet-4-5-20250929         # 使用的模型
  night_start_hour: 1                       # 夜间开始时间（凌晨 1 点）
  night_end_hour: 9                         # 夜间结束时间（上午 9 点）
  threshold_day: 20.0                       # 白天使用阈值（95%）
  threshold_night: 80.0                     # 夜间使用阈值（96%）
  usage_command: claude /usage              # 使用情况查询命令

# 代理配置
agent:
  workspace_root: ./workspace               # 工作空间根目录
  task_timeout_seconds: 1800                # 任务超时（30 分钟）
  max_concurrent_tasks: 1                   # 最大并发任务数

# Git 配置
git:
  enabled: false                             # 是否启用 Git 集成
  use_remote_repo: true                     # 是否使用远程仓库
  remote_repo_url: git@github.com:user/repo.git  # 远程仓库 URL
  auto_create_repo: true                    # 自动创建仓库（如果不存在）
  thought_branch: thought-ideas             # 随机想法分支名
  auto_commit: true                         # 自动提交
  auto_push: true                           # 自动推送
  auto_pr: true                             # 自动创建 PR（严肃任务）

# 多代理工作流配置
multi_agent_workflow:
  enabled: true                             # 是否启用多代理工作流

  planner:
    enabled: true                           # 是否启用 Planner Agent
    max_turns: 10                            # 最大轮次
    prompt_template: |                      # Planner 提示模板
      You are a planning agent. Analyze the following task and create a detailed execution plan:

      Task: {task_description}

      Provide:
      1. Step-by-step plan
      2. Potential challenges
      3. Required resources

  worker:
    enabled: true                           # 是否启用 Worker Agent
    max_turns: 30                            # 最大轮次
    prompt_template: |                      # Worker 提示模板
      You are a worker agent. Execute the following task based on the plan:

      Task: {task_description}
      Plan: {plan}

      Complete the task and provide results.

  evaluator:
    enabled: true                           # 是否启用 Evaluator Agent
    max_turns: 10                            # 最大轮次
    prompt_template: |                      # Evaluator 提示模板
      You are an evaluator agent. Review the completed work:

      Task: {task_description}
      Result: {result}

      Validate:
      1. Correctness
      2. Completeness
      3. Code quality

# 任务自动生成配置
auto_generation:
  enabled: true                             # 是否启用自动任务生成
  interval_seconds: 300                     # 生成间隔（5 分钟）
  max_generated_tasks: 10                   # 最大自动生成任务数

  strategies:                               # 生成策略（权重总和应为 100）
    refine_focused: 45                      # 改进现有工作（45%）
    balanced: 35                            # 混合策略（35%）
    new_friendly: 20                        # 创建新项目（20%）

# 监控和日志配置
monitoring:
  log_level: INFO                           # 日志级别（DEBUG, INFO, WARNING, ERROR）
  log_file: workspace/data/agent.log        # 日志文件路径
  metrics_file: workspace/data/metrics.jsonl  # 指标文件路径

  daily_report:
    enabled: true                           # 是否生成日报
    time: "09:00"                           # 生成时间（每天上午 9 点）
    slack_channel: "#sleepless-reports"     # 发送到的 Slack 频道

# Slack 配置（通过环境变量）
# SLACK_BOT_TOKEN, SLACK_APP_TOKEN

# 数据库配置
database:
  path: workspace/data/tasks.db             # SQLite 数据库路径
  backup_enabled: true                      # 是否启用自动备份
  backup_interval_hours: 24                 # 备份间隔（24 小时）
```

### 环境变量

```bash
# 必需
SLACK_BOT_TOKEN=xoxb-...                    # Slack Bot 令牌
SLACK_APP_TOKEN=xapp-...                    # Slack App 令牌

# 可选（覆盖 config.yaml）
AGENT_WORKSPACE_ROOT=./workspace            # 工作空间根目录
AGENT_DB_PATH=./workspace/data/tasks.db     # 数据库路径
AGENT_RESULTS_PATH=./workspace/data/results # 结果目录
LOG_LEVEL=INFO                              # 日志级别
SLEEPLESS_LOG_LEVEL=DEBUG                   # Sleepless 专用日志级别

# Claude Code（如果需要覆盖）
CLAUDE_BINARY_PATH=claude                   # Claude CLI 路径
CLAUDE_MODEL=claude-sonnet-4-5-20250929     # 模型名称
```

### 工作空间结构详解

```
workspace/
├── data/                                   # 持久化数据（系统）
│   ├── tasks.db                            # SQLite 数据库
│   │   └── 表：tasks, results, metadata
│   │
│   ├── results/                            # 任务结果 JSON
│   │   ├── task_1.json
│   │   ├── task_2.json
│   │   └── ...
│   │
│   ├── reports/                            # 日报 Markdown
│   │   ├── 2025-10-22.md
│   │   ├── 2025-10-23.md
│   │   └── ...
│   │
│   ├── agent.log                           # 应用日志
│   ├── metrics.jsonl                       # 性能指标（每行一个 JSON）
│   └── chat_sessions.json                  # 聊天会话状态
│
├── tasks/                                  # 任务工作空间（隔离）
│   ├── task_1/                             # 任务 1 的工作目录
│   │   ├── main.py                         # 生成的代码文件
│   │   ├── .claude/                        # Claude 元数据
│   │   └── ...
│   ├── task_2/
│   └── ...
│
├── projects/                               # 项目工作空间（严肃任务）
│   ├── backend/                            # 项目名称
│   │   ├── src/
│   │   ├── tests/
│   │   ├── .git/                           # 项目的 Git 仓库
│   │   └── README.md
│   ├── frontend/
│   └── ...
│
├── shared/                                 # 跨任务共享文件
│   ├── libraries/                          # 共享库
│   ├── templates/                          # 代码模板
│   └── utils/                              # 工具脚本
│
└── trash/                                  # 软删除的项目
    ├── backend_deleted_2025-10-22/
    └── ...
```

---

## 关键功能实现原理

### 1. 24/7 守护进程机制

**实现原理：**

守护进程基于 Python 的 `asyncio` 事件循环实现，使用以下技术：

```python
import asyncio
import signal

class SleeplessAgent:
    def __init__(self):
        self.running = True
        self.setup_signal_handlers()

    def setup_signal_handlers(self):
        """设置信号处理器以支持优雅关闭"""
        signal.signal(signal.SIGTERM, self.handle_shutdown)
        signal.signal(signal.SIGINT, self.handle_shutdown)

    def handle_shutdown(self, signum, frame):
        """处理关闭信号"""
        logger.info(f"Received signal {signum}, shutting down gracefully...")
        self.running = False

    async def run(self):
        """主事件循环"""
        logger.info("Sleepless Agent starting...")

        # 启动子系统
        await self.start_slack_bot()

        # 主循环
        while self.running:
            try:
                # 1. 检查使用配额
                if not self.scheduler.should_generate_tasks():
                    await asyncio.sleep(60)
                    continue

                # 2. 处理任务
                await self.process_next_task()

                # 3. 自动生成任务
                if self.should_auto_generate():
                    await self.auto_generate_task()

                # 4. 生成报告
                if self.is_report_time():
                    await self.generate_daily_report()

                # 短暂休眠
                await asyncio.sleep(10)

            except Exception as e:
                logger.error(f"Error in main loop: {e}")
                await asyncio.sleep(60)  # 错误后等待

        # 清理
        await self.cleanup()
        logger.info("Sleepless Agent stopped")

# 部署为 systemd 服务（Linux）
[Unit]
Description=Sleepless Agent - 24/7 AI Assistant
After=network.target

[Service]
Type=simple
User=sleepless
WorkingDirectory=/opt/sleepless-agent
ExecStart=/opt/sleepless-agent/venv/bin/python -m sleepless_agent.core.daemon
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**关键特性：**
- **持续运行**：无限循环 + 优雅关闭
- **错误恢复**：捕获异常并自动重试
- **资源管理**：定期清理临时文件和日志轮转
- **自动重启**：通过 systemd 或 launchd 实现崩溃后重启

---

### 2. 智能调度算法（基于时间和配额）

**核心逻辑：**

```python
class SmartScheduler:
    def should_generate_tasks(self) -> bool:
        """决定是否应该生成新任务"""
        # 1. 获取当前使用情况
        usage_percent = self.budget_manager.get_current_usage()

        # 2. 获取当前适用的阈值
        threshold = self._get_time_based_threshold()

        # 3. 比较
        if usage_percent >= threshold:
            logger.info(f"Usage {usage_percent}% >= threshold {threshold}%, pausing task generation")
            return False

        return True

    def _get_time_based_threshold(self) -> float:
        """根据当前时间返回适当的阈值"""
        current_hour = datetime.now().hour

        # 夜间（1:00 - 9:00）
        if self.config['night_start_hour'] <= current_hour < self.config['night_end_hour']:
            return self.config['threshold_night']  # 96% - 更激进

        # 白天（9:00 - 1:00）
        else:
            return self.config['threshold_day']    # 95% - 保守

    def get_next_task(self) -> Optional[Task]:
        """选择下一个要执行的任务（优先级调度）"""
        # 优先级顺序：SERIOUS > THOUGHT > GENERATED

        # 1. 首先尝试获取严肃任务
        task = self.task_queue.get_task_by_priority(TaskPriority.SERIOUS)
        if task:
            return task

        # 2. 然后是随机想法
        task = self.task_queue.get_task_by_priority(TaskPriority.THOUGHT)
        if task:
            return task

        # 3. 最后是自动生成的任务
        task = self.task_queue.get_task_by_priority(TaskPriority.GENERATED)
        return task
```

**时间窗口优化：**
- **夜间（1 AM - 9 AM）**：96% 阈值，充分利用睡眠时间
- **白天（9 AM - 1 AM）**：95% 阈值，为手动使用保留容量

**使用监控：**
```python
def get_current_usage(self) -> float:
    """通过 Claude CLI 获取使用情况"""
    result = subprocess.run(
        ['claude', '/usage'],
        capture_output=True,
        text=True,
        timeout=10
    )

    # 解析输出（示例格式："Usage: 1234/5000 (24.68%)"）
    match = re.search(r'(\d+\.\d+)%', result.stdout)
    if match:
        return float(match.group(1))

    return 0.0
```

---

### 3. 多代理工作流（Planner/Worker/Evaluator）

**三阶段执行流程：**

```python
class MultiAgentExecutor:
    async def execute_with_workflow(self, task: Task) -> Result:
        """使用多代理工作流执行任务"""

        # Phase 1: Planner Agent
        plan = await self._plan_phase(task)

        # Phase 2: Worker Agent
        result = await self._work_phase(task, plan)

        # Phase 3: Evaluator Agent
        evaluation = await self._eval_phase(task, result)

        # 根据评估决定是否接受结果
        if evaluation['passed']:
            return result
        else:
            # 如果评估失败，可以重试或标记为失败
            logger.warning(f"Task {task.id} failed evaluation: {evaluation['reason']}")
            return Result(status='failed', error=evaluation['reason'])

    async def _plan_phase(self, task: Task) -> str:
        """Planner Agent：分析任务并创建计划"""
        prompt = self.config['planner']['prompt_template'].format(
            task_description=task.description
        )

        response = await self.claude_sdk.query(
            prompt=prompt,
            max_turns=self.config['planner']['max_turns']
        )

        return response  # 返回计划文本

    async def _work_phase(self, task: Task, plan: str) -> Result:
        """Worker Agent：根据计划执行任务"""
        prompt = self.config['worker']['prompt_template'].format(
            task_description=task.description,
            plan=plan
        )

        response = await self.claude_sdk.query(
            prompt=prompt,
            max_turns=self.config['worker']['max_turns'],
            workspace=self._get_workspace(task)
        )

        return Result(
            status='completed',
            output=response,
            workspace=self._get_workspace(task)
        )

    async def _eval_phase(self, task: Task, result: Result) -> dict:
        """Evaluator Agent：验证完成的工作"""
        prompt = self.config['evaluator']['prompt_template'].format(
            task_description=task.description,
            result=result.output
        )

        response = await self.claude_sdk.query(
            prompt=prompt,
            max_turns=self.config['evaluator']['max_turns']
        )

        # 解析评估结果
        return self._parse_evaluation(response)
```

**角色专业化：**
- **Planner**：专注于分析和规划，不执行代码
- **Worker**：专注于执行，可以使用所有工具（Edit, Write, Bash 等）
- **Evaluator**：专注于验证，检查正确性、完整性和质量

---

### 4. 自动任务生成策略

**三种生成策略：**

```python
class AutoTaskGenerator:
    def __init__(self, config: dict):
        self.strategies = {
            'refine_focused': self._refine_focused_strategy,
            'balanced': self._balanced_strategy,
            'new_friendly': self._new_friendly_strategy
        }
        self.weights = config['auto_generation']['strategies']

    async def generate_task(self) -> Optional[Task]:
        """根据权重随机选择策略并生成任务"""
        strategy = self._weighted_choice(self.weights)
        return await self.strategies[strategy]()

    async def _refine_focused_strategy(self) -> Task:
        """改进现有工作（45% 权重）"""
        # 1. 查找最近的未完成任务
        recent_tasks = self.task_queue.get_recent_tasks(limit=10)

        # 2. 分析哪些任务可以改进
        for task in recent_tasks:
            if self._can_be_refined(task):
                return Task(
                    description=f"[REFINE:#{task.id}] Improve and complete {task.description}",
                    priority=TaskPriority.GENERATED,
                    task_type=TaskType.REFINE,
                    refine_task_id=task.id
                )

        return None

    async def _balanced_strategy(self) -> Task:
        """混合策略（35% 权重）"""
        # 50% 机会改进，50% 机会创建新任务
        if random.random() < 0.5:
            return await self._refine_focused_strategy()
        else:
            return await self._new_friendly_strategy()

    async def _new_friendly_strategy(self) -> Task:
        """创建新项目（20% 权重）"""
        # 1. 从 Claude 获取创意
        prompt = """
        Suggest a new, innovative project idea that would be useful.
        Focus on:
        - Practical utility
        - Novelty
        - Feasibility

        Provide a one-sentence description.
        """

        response = await self.claude_sdk.query(prompt=prompt, max_turns=1)

        # 2. 创建任务
        return Task(
            description=f"[NEW] {response}",
            priority=TaskPriority.GENERATED,
            task_type=TaskType.NEW
        )

    def _weighted_choice(self, weights: dict) -> str:
        """根据权重随机选择"""
        choices = list(weights.keys())
        weights_list = list(weights.values())
        return random.choices(choices, weights=weights_list)[0]
```

**工作空间策略：**
- **NEW 任务**：创建新的隔离工作空间 `workspace/tasks/<task_id>/`
- **REFINE 任务**：重用原始任务的工作空间，保持连续性

---

## 开发指南

### 如何添加新的 Slack 命令

**步骤：**

1. **在 Slack App 配置中添加命令**
   - 访问 https://api.slack.com/apps/your-app/slash-commands
   - 点击 "Create New Command"
   - 设置命令名称（如 `/analyze`）和描述

2. **在 `interfaces/bot.py` 中添加处理器**

```python
@app.command("/analyze")
def handle_analyze(ack, command, client):
    """处理 /analyze 命令"""
    ack()  # 立即确认

    # 解析参数
    text = command['text']

    # 执行逻辑
    try:
        result = analyzer.analyze(text)

        # 回复用户
        client.chat_postMessage(
            channel=command['channel_id'],
            text=f"✅ 分析完成：\n{result}"
        )
    except Exception as e:
        client.chat_postMessage(
            channel=command['channel_id'],
            text=f"❌ 错误：{str(e)}"
        )
```

3. **在 `interfaces/cli.py` 中添加 CLI 命令（可选）**

```python
@click.command()
@click.argument('target')
def analyze(target):
    """分析指定目标"""
    result = analyzer.analyze(target)
    console.print(result)

cli.add_command(analyze)
```

---

### 如何扩展任务类型

**步骤：**

1. **在 `core/models.py` 中添加新的枚举值**

```python
class TaskType(enum.Enum):
    NEW = "new"
    REFINE = "refine"
    ANALYZE = "analyze"  # 新增
```

2. **在 `core/executor.py` 中处理新类型**

```python
async def execute(self, task: Task) -> Result:
    if task.task_type == TaskType.ANALYZE:
        return await self._execute_analyze_task(task)
    # ... 其他类型
```

3. **更新任务创建逻辑**

```python
# 在命令处理器中
task = Task(
    description=description,
    task_type=TaskType.ANALYZE,
    priority=priority
)
```

---

### 如何自定义调度策略

**步骤：**

1. **创建自定义调度器类**

```python
from sleepless_agent.scheduling.scheduler import SmartScheduler

class CustomScheduler(SmartScheduler):
    def should_generate_tasks(self) -> bool:
        """自定义逻辑"""
        # 例如：只在特定日期生成任务
        if datetime.now().weekday() in [5, 6]:  # 周末
            return False

        return super().should_generate_tasks()

    def get_next_task(self) -> Optional[Task]:
        """自定义任务选择逻辑"""
        # 例如：优先处理特定项目的任务
        task = self.task_queue.get_task_by_project("critical-project")
        if task:
            return task

        return super().get_next_task()
```

2. **在 `daemon.py` 中使用自定义调度器**

```python
from my_module import CustomScheduler

class SleeplessAgent:
    def __init__(self, config):
        # 使用自定义调度器
        self.scheduler = CustomScheduler(config)
```

---

## 技术亮点和创新点

### 1. 工作空间隔离设计

**问题：** 如何安全地并行执行多个 AI 任务而不互相干扰？

**解决方案：**
- 每个任务在独立目录中执行
- 严格的路径权限控制
- 共享资源通过 `workspace/shared/` 目录
- 系统目录（`data/`）受保护，任务无法访问

**实现细节：**

```python
class WorkspaceManager:
    def setup_task_workspace(self, task: Task) -> str:
        """为任务设置隔离工作空间"""
        if task.project_name:
            # 严肃任务 -> 项目工作空间
            workspace = f"workspace/projects/{task.project_name}"
        else:
            # 随机想法 -> 任务工作空间
            workspace = f"workspace/tasks/task_{task.id}"

        # 创建目录
        os.makedirs(workspace, exist_ok=True)

        # 设置权限（只读共享，读写自己）
        self._setup_permissions(workspace)

        # 创建符号链接到共享资源
        os.symlink(
            "workspace/shared",
            f"{workspace}/shared",
            target_is_directory=True
        )

        return workspace

    def _setup_permissions(self, workspace: str):
        """设置目录权限"""
        # 任务只能访问：
        # - 自己的工作空间（读写）
        # - shared/ 目录（只读）
        # - 不能访问 data/ 目录
        pass  # 实际实现使用文件系统权限或 Docker 容器
```

**优势：**
- **安全性**：任务无法访问其他任务的数据
- **可靠性**：一个任务的崩溃不影响其他任务
- **可扩展性**：理论上可以并行执行无限个任务

---

### 2. Claude Code SDK 集成方式

**创新点：** 不直接调用 Claude API，而是通过 Claude Code CLI 和 Python Agent SDK

**优势：**
1. **工具生态系统**：Claude Code 内置了丰富的工具（Edit, Write, Bash, Grep 等）
2. **上下文管理**：CLI 自动处理对话历史和上下文窗口
3. **配额统一**：使用 Claude Code Pro 计划的配额，无需单独的 API 密钥
4. **本地优先**：所有操作在本地执行，数据不离开机器

**实现示例：**

```python
from claude_agent_sdk import ClaudeAgentSDK

class ClaudeCodeExecutor:
    def __init__(self, config: dict):
        self.sdk = ClaudeAgentSDK(
            binary_path=config['claude_code']['binary_path'],
            model=config['claude_code']['model']
        )

    async def execute(self, task: Task) -> Result:
        """使用 SDK 执行任务"""
        response = await self.sdk.query(
            prompt=task.description,
            workspace=self._get_workspace(task),
            max_turns=10,  # 支持多轮对话
            tools=['edit', 'write', 'bash', 'grep', 'read']
        )

        return self._parse_response(response)
```

---

### 3. 智能配额管理

**创新点：** 时间感知的使用阈值

**为什么重要：**
- Claude Code Pro 计划有每日使用限制
- 需要平衡自主性和保留手动使用的容量

**策略：**
- **夜间（1 AM - 9 AM）**：96% 阈值 - 用户睡觉时充分利用
- **白天（9 AM - 1 AM）**：95% 阈值 - 为用户手动使用保留 5%

**动态调整：**

```python
def get_adaptive_threshold(self) -> float:
    """根据历史使用模式自适应调整阈值"""
    # 分析过去 7 天的使用模式
    history = self.usage_history.get_last_days(7)

    # 如果用户白天很少手动使用，可以提高白天阈值
    avg_manual_usage = self._calculate_manual_usage(history)

    if avg_manual_usage < 10:  # 用户很少手动使用
        return self.threshold_day + 2  # 提高到 97%
    else:
        return self.threshold_day  # 保持 95%
```

---

## 总结

Sleepless Agent 是一个精心设计的 24/7 AI 代理系统，具有以下核心优势：

1. **完全自主**：无需人工干预即可持续运行
2. **智能调度**：基于时间和配额的优化任务执行
3. **安全隔离**：每个任务在独立工作空间中执行
4. **深度集成**：与 Slack、Git、Claude Code 无缝集成
5. **可扩展性**：模块化设计，易于添加新功能

**适用场景：**
- 个人开发者：自动化重复性开发任务
- 团队协作：通过 Slack 集中管理 AI 任务
- 研究探索：利用空闲时间探索新想法
- 持续集成：自动代码审查、测试、文档生成

**技术栈总结：**
- Python 3.11+ | Claude Code CLI | Slack Bolt SDK
- SQLAlchemy + SQLite | GitPython | Rich

**项目链接：**
- GitHub: https://github.com/context-machine-lab/sleepless-agent
- 文档: https://context-machine-lab.github.io/sleepless-agent/
- PyPI: https://pypi.org/project/sleepless-agent/

---

*最后更新：2025-12-31*
*文档版本：1.0*
