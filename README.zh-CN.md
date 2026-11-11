<p align="center">
  <a href="https://tauric.ai/">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="assets/Trading-R1-dark.svg">
      <img src="assets/Trading-R1.svg" alt="Trading-R1" width="60%">
    </picture>
  </a>
</p>

<p align="center">
  <a href="https://arxiv.org/abs/2509.11420"><img alt="arXiv" src="https://img.shields.io/badge/arXiv-2509.11420-B31B1B?logo=arxiv"/></a>
  <a href="https://discord.com/invite/hk9PGKShPK"><img alt="Discord" src="https://img.shields.io/badge/Discord-TauricResearch-7289da?logo=discord&amp;logoColor=white"/></a>
  <a href="https://x.com/TauricResearch"><img alt="X" src="https://img.shields.io/badge/X-TauricResearch-white?logo=x&amp;logoColor=white"/></a>
  <a href="https://github.com/TauricResearch/"><img alt="GitHub" src="https://img.shields.io/badge/GitHub-TauricResearch-14C290?logo=github&amp;logoColor=white"/></a>
  <a href="https://tauric.ai/"><img alt="Website" src="https://img.shields.io/badge/Website-tauric.ai-14C290?logo=data:image/svg%2bxml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgc3Ryb2tlPSIjZmZmIiBzdHJva2Utd2lkdGg9IjIiPjxjaXJjbGUgY3g9IjEyIiBjeT0iMTIiIHI9IjEwIi8+PHBhdGggZD0iTTIgMTJoMjBNMTIgMmExNSAxNSAwIDAgMSAwIDIwTTEyIDJhMTUgMTUgMCAwIDAgMCAyMCIvPjwvc3ZnPg=="/></a>
</p>

<p align="center">
  <a href="https://www.readme-i18n.com/TauricResearch/Trading-R1?lang=de">Deutsch</a> |
  <a href="https://www.readme-i18n.com/TauricResearch/Trading-R1?lang=es">Español</a> |
  <a href="https://www.readme-i18n.com/TauricResearch/Trading-R1?lang=fr">français</a> |
  <a href="https://www.readme-i18n.com/TauricResearch/Trading-R1?lang=ja">日本語</a> |
  <a href="https://www.readme-i18n.com/TauricResearch/Trading-R1?lang=ko">한국어</a> |
  <a href="https://www.readme-i18n.com/TauricResearch/Trading-R1?lang=pt">Português</a> |
  <a href="https://www.readme-i18n.com/TauricResearch/Trading-R1?lang=ru">Русский</a> |
  <a href="https://www.readme-i18n.com/TauricResearch/Trading-R1?lang=zh">中文</a>
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README.zh-CN.md">简体中文</a>
</p>

<h2 align="center">Trading-R1：用于金融研究与分析的终端</h2>

<p align="center">
  <img src="assets/trading-r1.gif" alt="Trading-R1 在终端中回答一个研究问题" width="100%">
</p>

## 快速开始

**1. 创建 Tauric 账户并获取 API 密钥。** [注册](https://tauric.ai/signup)，然后[创建密钥](https://tauric.ai/settings/api-keys)。

**2. 安装。**

```bash
curl -fsSL https://raw.githubusercontent.com/TauricResearch/Trading-R1/main/install.sh | sh
```

或者，如果你有 Python 3.12 或更高版本以及 `bun`：`pip install trading-r1`。

**3. 添加密钥并启动。**

```bash
export TAURIC_API_KEY=your-key
trading-r1
```

如果想把密钥保存下来，改为运行 `trading-r1 secrets set TAURIC_API_KEY`，或者在应用中按 `/` 并选择 *API keys…*。

要卸载，如果之前安装了后台服务，先运行 `trading-r1 daemon uninstall`，然后运行 `uv tool uninstall trading-r1` 或 `pip uninstall trading-r1`。你的数据保留在 `~/.trading-r1` 和你的报告目录中。

## 选择模型

| 名称 | 模型 | 需要 |
|---|---|---|
| `tauric` | `tauric` | `TAURIC_API_KEY` |
| `gpt` | `gpt-6.1-sol` | `OPENAI_API_KEY` |
| `anthropic` | `claude-sonnet-5-5` | `ANTHROPIC_API_KEY` |
| `gemini` | `gemini-3.8-flash` | `GEMINI_API_KEY` |
| `vllm` | `Qwen3.8-27B` | 你自己运行的 vLLM 服务器 |

应用会从你已配置好的第一个模型启动，优先顺序为 OpenAI、Anthropic、Gemini、Tauric、vLLM。可以在命令面板中用 *Model…* 更换，或者用 `trading-r1 --profile gpt` 以指定模型启动。要把 `vllm` 指向你自己的服务器，请参阅[配置](MANUAL.md#configuration)。

## 使用

```bash
trading-r1                                          # open the terminal
trading-r1 "What's NVDA's last quarterly revenue?"  # and ask a first question
trading-r1 --mode analytical                        # start in a mode
```

| 按键 | 作用 |
|---|---|
| `/` 或 `^K` | 打开命令面板 |
| `shift+tab` | 切换模式 |
| `^N` | 开始新的研究 |
| `esc` | 取消正在运行的这一轮，或关闭对话框 |
| `^C` | 退出 |

| 模式 | 侧重 |
|---|---|
| `auto` | 全套工具，由智能体自行决定 |
| `research` | 优先联网，适合深度研究 |
| `analytical` | 优先数字：数据与估值工具，联网用于补充背景 |

命令面板列出所有操作：会话、模型与模式、报告、渠道、定时报告、工具与技能、API 密钥、后台服务、Settings 和 Help。

## 架构

<p align="center">
  <img src="assets/taxonomy.svg" alt="Trading-R1 的工具：始终加载的工具、按学科和领域划分的金融工具，以及工作流技能" width="100%">
</p>

## 报告与发送

*Export report…* 会把上一条回答保存为 PDF，并且默认同时保存为 Markdown，存放在 `~/Documents/Trading-R1` 或 `~/Trading-R1`。

*Send report…* 会把它发送到你的渠道；如果还没有配置渠道，它会引导你完成设置。

| 渠道 | 你提供 | 得到 |
|---|---|---|
| `telegram` | 来自 @BotFather 的机器人令牌，然后给机器人发一条消息并确认对话 | 文本和 PDF |
| `email` | 你的邮箱地址和应用专用密码（预设：`gmail`、`icloud`、`yahoo`、`fastmail`） | 文本和 PDF |
| `folder` | 一个路径 | 报告文件 |

要能在手机上提问，把 Telegram 渠道的 `reply` 设为 `on`。在后端运行期间，你的机器人会用一次简短的研究来回答你的消息，`/new` 则重新开始。

## 定时报告

定时任务会按计划提出一个问题，并把报告发送到你的渠道。使用命令面板中的 *Scheduled reports…*，或者：

```bash
trading-r1 jobs add --name "Morning brief" --every weekdays --time 08:00 --tz America/New_York \
    --prompt "What moved my watchlist overnight? NVDA, AAPL, MSFT." --deliver email,telegram
```

报告、渠道和定时任务的命令与选项请见[用户手册](MANUAL.md#reports-channels-and-jobs)。

## 后台服务

定时任务和 Telegram 回复需要有一个运行中的后端，而终端自带的后端会随你关闭终端而停止。要让它持续运行，可以安装一个用户级服务（Linux 上用 systemd，macOS 上用 launchd），在命令面板中选择 *Background service…*，或者：

```bash
trading-r1 daemon install      # also: status, logs, start, stop, restart, uninstall
```

更新之后，或者修改了 shell 配置文件里的密钥之后，再运行一次 `trading-r1 daemon install`。

## 用户手册

命令、设置、会话同步、你的数据和密钥、配置以及添加技能，都在[用户手册](MANUAL.md)中。如果要参与代码开发，请参阅[贡献指南](CONTRIBUTING.md)。

## 引用

```bibtex
@article{xiao2025trading,
  title={Trading-r1: Financial trading with llm reasoning via reinforcement learning},
  author={Xiao, Yijia and Sun, Edward and Chen, Tong and Wu, Fang and Luo, Di and Wang, Wei},
  journal={arXiv preprint arXiv:2509.11420},
  year={2025}
}
```

## 免责声明

仅供研究与教育使用，不构成投资建议。

## 许可证

Apache-2.0。参见 [LICENSE](LICENSE)。

<!-- 在 PyPI 页面上本文件的图片可能无法显示，
     请到 GitHub 仓库阅读完整版本。 -->
