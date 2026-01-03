# Sleepless Agent 实施计划

## 📋 概述

本实施计划旨在扩展 Sleepless Agent 的功能，支持更灵活的部署和使用方式。

### 核心目标

1. **支持 API Key 认证**：允许通过 API Key 配置 Claude Code CLI，而不仅限于订阅账号
2. **自定义使用配额**：为 API Key 模式提供可配置的配额管理
3. **多平台接口支持**：支持命令行、Slack、飞书、企业微信等多种交互方式

---

## 🎯 解决方案概览

### 方案一：API Key 认证支持（预计 0.5-1 天）

**目标**：支持通过环境变量配置 Claude Code CLI 使用 API Key 认证

**技术方案**：
- 利用 Claude Code CLI 原生支持的环境变量 `ANTHROPIC_AUTH_TOKEN`
- 在执行器初始化时设置相应的环境变量
- 支持静态 API Key、动态 API Key（通过辅助脚本）和 LLM Gateway

**关键修改**：
- `src/sleepless_agent/config.yaml` - 添加认证模式配置
- `src/sleepless_agent/core/executor.py` - 添加环境变量设置逻辑
- `.env.example` - 更新环境变量示例
- `docs/api-key-setup.md` - 新建 API Key 配置指南

### 方案二：自定义配额管理（预计 1-2 天）

**目标**：为 API Key 模式提供本地配额追踪和限制

**技术方案**：
- 创建 `LocalQuotaTracker` 类，追踪请求数、输入/输出 token 数
- 在 `ProPlanUsageChecker` 中集成本地追踪器
- 在任务执行后自动记录使用情况
- 支持每日配额重置（可配置重置时间）

**关键修改**：
- `src/sleepless_agent/monitoring/local_quota_tracker.py` - 新建本地配额追踪器
- `src/sleepless_agent/monitoring/pro_plan_usage.py` - 集成本地追踪
- `src/sleepless_agent/core/executor.py` - 记录使用
- `src/sleepless_agent/config.yaml` - 添加配额配置

### 方案三：多平台接口支持（预计 5-7 天）

**目标**：重构接口层，支持多种交互平台

**技术方案**：
- 实现三层架构：Handler（业务逻辑）→ Adapter（格式化）→ Interface（平台 SDK）
- 提取公共业务逻辑到 Handler 层
- 创建平台特定的 Adapter（Slack、CLI、飞书等）
- 重构现有 Slack 和 CLI 接口
- 添加交互式 CLI 和飞书支持

**关键修改**：
- `src/sleepless_agent/handlers/` - 新建 Handler 层
- `src/sleepless_agent/adapters/` - 新建 Adapter 层
- `src/sleepless_agent/interfaces/bot.py` - 重构使用新架构
- `src/sleepless_agent/interfaces/cli.py` - 重构使用新架构
- `src/sleepless_agent/interfaces/interactive_cli.py` - 新建交互式 CLI
- `src/sleepless_agent/interfaces/feishu.py` - 新建飞书接口

---

## 📅 实施阶段

### 阶段一：API Key 认证支持（已完成准备工作）

**步骤 1.1：配置文件更新**

在 `src/sleepless_agent/config.yaml` 添加：

```yaml
claude_code:
  # 认证模式：'subscription'（订阅账号）或 'api_key'（API Key）
  auth_mode: subscription

  # API Key 认证配置
  api_key: ${ANTHROPIC_AUTH_TOKEN}  # 从环境变量读取
  api_key_helper: null  # 动态密钥辅助脚本路径（可选）
  api_key_helper_ttl_ms: 3600000  # 密钥刷新间隔（默认 1 小时）
  base_url: null  # LLM Gateway URL（可选）

  # 现有配置保持不变
  binary_path: claude
  model: claude-sonnet-4-5-20250929
  threshold_day: 20.0
  threshold_night: 80.0
```

**步骤 1.2：执行器修改**

在 `src/sleepless_agent/core/executor.py` 添加认证设置方法：

```python
def _setup_authentication(self):
    """根据配置设置 Claude Code CLI 认证"""
    auth_mode = self.config.get('auth_mode', 'subscription')

    if auth_mode == 'api_key':
        # 设置 API Key
        if self.config.get('api_key'):
            os.environ['ANTHROPIC_AUTH_TOKEN'] = self.config['api_key']

        # 设置 LLM Gateway（如果配置）
        if self.config.get('base_url'):
            os.environ['ANTHROPIC_BASE_URL'] = self.config['base_url']
```

**步骤 1.3：环境变量示例**

更新 `.env.example`：

```bash
# Slack Bot（可选 - 如果使用 Slack 接口）
# SLACK_BOT_TOKEN=xoxb-your-slack-bot-token
# SLACK_APP_TOKEN=xapp-your-slack-app-token

# Claude Code 认证
# 方式 1: 订阅账号（需要先运行 claude login）
# 无需设置环境变量

# 方式 2: API Key 认证
# ANTHROPIC_AUTH_TOKEN=sk-ant-your-api-key

# 可选：LLM Gateway
# ANTHROPIC_BASE_URL=https://your-litellm-server:4000

# Agent 配置
AGENT_WORKSPACE_ROOT=./workspace
AGENT_DB_PATH=./workspace/data/tasks.db
AGENT_RESULTS_PATH=./workspace/data/results
```

**步骤 1.4：文档编写**

创建 `docs/api-key-setup.md` 详细说明配置方法。

### 阶段二：自定义配额管理（待实施）

**步骤 2.1：本地配额追踪器**

创建 `src/sleepless_agent/monitoring/local_quota_tracker.py`：

```python
class LocalQuotaTracker:
    """本地配额追踪器，用于 API Key 模式的配额管理"""

    def __init__(self, config: dict, storage_path: Path):
        self.config = config  # daily_requests, daily_input_tokens, daily_output_tokens
        self.storage_path = storage_path
        self.data = self._load_data()

    def add_usage(self, input_tokens: int, output_tokens: int):
        """记录一次使用"""
        # 实现配额追踪逻辑

    def get_current_usage(self) -> Tuple[float, Optional[datetime]]:
        """返回：(使用百分比, 重置时间)"""
        # 计算三个维度的使用百分比，返回最高值
```

**步骤 2.2：集成到配额检查器**

修改 `src/sleepless_agent/monitoring/pro_plan_usage.py`。

**步骤 2.3：执行器记录使用**

在任务完成后记录 token 使用情况。

### 阶段三：多平台接口支持（待实施）

**步骤 3.1：Handler 层**

创建业务逻辑层：
- `handlers/base.py` - 基类和 `CommandResult` 数据类
- `handlers/task_handler.py` - 任务管理逻辑
- `handlers/check_handler.py` - 状态检查逻辑
- `handlers/report_handler.py` - 报告生成逻辑

**步骤 3.2：Adapter 层**

创建格式化适配器：
- `adapters/base.py` - 适配器基类
- `adapters/slack_adapter.py` - Slack Block Kit 格式化
- `adapters/cli_adapter.py` - 命令行格式化（Rich）
- `adapters/feishu_adapter.py` - 飞书卡片格式化

**步骤 3.3：重构现有接口**

使用新架构重构 `interfaces/bot.py` 和 `interfaces/cli.py`。

**步骤 3.4：新接口实现**

- `interfaces/interactive_cli.py` - 交互式命令行（REPL）
- `interfaces/feishu.py` - 飞书机器人

---

## 🔧 配置示例

### API Key 模式配置

```yaml
# config.yaml
claude_code:
  auth_mode: api_key
  api_key: ${ANTHROPIC_AUTH_TOKEN}
  model: claude-sonnet-4-5-20250929
  threshold_day: 20.0
  threshold_night: 80.0
```

```bash
# .env
ANTHROPIC_AUTH_TOKEN=sk-ant-your-api-key
```

### 自定义配额配置

```yaml
# config.yaml
claude_code:
  auth_mode: api_key

  custom_quota:
    enabled: true
    daily_requests: 1000
    daily_input_tokens: 500000
    daily_output_tokens: 500000
    reset_hour: 0  # UTC 0 点重置

  threshold_day: 20.0
  threshold_night: 80.0
```

### 订阅模式配置（现有方式，保持兼容）

```yaml
# config.yaml
claude_code:
  auth_mode: subscription
  binary_path: claude
  usage_command: claude /usage
  model: claude-sonnet-4-5-20250929
  threshold_day: 20.0
  threshold_night: 80.0
```

---

## ✅ 验收标准

### 功能性
- ✅ 支持订阅账号和 API Key 两种认证方式，可通过配置切换
- ✅ 支持自定义配额管理，自动追踪使用并在达到阈值时暂停
- ✅ 交互式 CLI 支持实时任务监控
- ✅ 飞书 Bot 支持核心命令（think, check, report 等）
- ✅ 所有现有功能保持正常工作

### 代码质量
- ✅ 业务逻辑和表现层分离
- ✅ 添加新接口只需约 300 行代码（vs 原来 1500+ 行）
- ✅ Handler 代码复用率 > 90%
- ✅ 配置文档完善，易于理解

### 文档完整性
- ✅ API Key 配置指南（`docs/api-key-setup.md`）
- ✅ 自定义配额使用说明
- ✅ 交互式 CLI 使用指南
- ✅ 飞书集成指南
- ✅ 更新 README.md 和 CLAUDE.md

---

## 📊 风险评估

### 低风险（方案一和二）
- **配置简单**：只需修改配置文件和环境变量
- **向后兼容**：现有订阅模式完全不受影响
- **官方支持**：环境变量配置是 Claude Code CLI 的官方推荐方式

### 中风险（方案三）
- **代码重构量大**：需要重构所有命令处理逻辑
- **测试覆盖**：需要确保重构后功能完全一致
- **新接口稳定性**：飞书等新接口需要充分测试

---

## 📝 实施时间表

| 阶段 | 预计时间 | 状态 |
|------|---------|------|
| 阶段一：API Key 认证支持 | 0.5-1 天 | 🔄 进行中 |
| 阶段二：自定义配额管理 | 1-2 天 | ⏳ 待开始 |
| 阶段三：多平台接口支持 | 5-7 天 | ⏳ 待开始 |
| **总计** | **6.5-10 天** | |

---

## 📚 参考资源

- [Claude Code CLI 文档](https://code.claude.com/)
- [LLM Gateway 配置指南](https://code.claude.com/docs/zh-CN/llm-gateway)
- [Anthropic API 文档](https://docs.anthropic.com/)
- [LiteLLM 文档](https://docs.litellm.ai/)
- [飞书开放平台](https://open.feishu.cn/)

---

## 🔄 更新日志

- **2026-01-01**：创建实施计划文档
- **2026-01-01**：开始阶段一 - API Key 认证支持

---

## 📧 联系方式

如有问题或建议，请通过以下方式联系：
- GitHub Issues: [sleepless-agent/issues](https://github.com/context-machine-lab/sleepless-agent/issues)
- 微信：见 [assets/wechat.png](assets/wechat.png)
- Discord: https://discord.gg/74my3Wkn
