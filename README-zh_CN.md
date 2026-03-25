<div align="center">

# 🛡 pre-push-check

**一个 Claude Code 技能，在推送代码到 GitHub 之前扫描仓库的安全问题、.gitignore 配置和 README 合规性。**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

[English](README.md) | [中文](README-zh_CN.md)

</div>

不要再推送密钥、遗漏 `.gitignore` 规则或提交不完整的 README 了。`pre-push-check` 是一个 [Claude Code 技能](https://docs.anthropic.com/en/docs/claude-code)，在代码公开之前充当质量门禁——系统性地检查三个方面，并提供自动修复。

## 特性

- 🔒 **密钥扫描** — 通过模式匹配检测 API Key、私钥、连接字符串和硬编码密码（AWS、OpenAI、GitHub PAT 等）
- 📁 **Gitignore 审计** — 根据项目类型（Node.js、Python、Rust、Go、Java）验证 `.gitignore` 覆盖范围，标记不应被追踪的文件
- 📝 **README 合规检查** — 检查双语内容（中英文）、必要章节和结构完整性
- 🔧 **自动修复** — 提供自动修复：添加 `.gitignore` 规则、移除被追踪的 `.env` 文件、生成缺失的 README 章节
- 📄 **README 生成器** — 从零生成专业的双语 README，包含徽章、特性列表和快速开始指南

## 两种模式

### 模式 A：推送前检查

在即将 push、ship 或创建 PR 时触发。执行全部三项检查并生成结构化报告：

1. **Gitignore 审计** — 扫描被追踪的文件，匹配已知模式（构建产物、IDE 文件、依赖目录、系统文件）
2. **敏感数据扫描** — 检查 `.env` 文件、含真实值的配置文件、API Key、私钥、连接字符串
3. **README 检查** — 双语内容、必要章节（标题、特性、快速开始）、推荐章节（徽章、架构、贡献指南）

### 模式 B：README 生成器

在要求写 README 或生成 README 时触发。扫描项目后生成 `README.md`（英文）+ `README-zh_CN.md`（中文），包含徽章、特性列表和快速开始。

## 快速开始

### 安装为 Claude Code 技能

```bash
# 克隆到技能目录
git clone https://github.com/royfhs/pre-push-check.git ~/.claude/skills/pre-push-check
```

### 使用方法

在 Claude Code 中直接说：

```
/pre-push-check
```

或者用自然语言触发：
- "Check the repo before I push"
- "帮我检查一下能不能推送"
- "Generate a README for this project"
- "帮我写个 README"

## 报告示例

```
## 推送前检查报告

### 1. Gitignore 配置
[通过] — .gitignore 覆盖完整

### 2. 敏感数据检查
[失败] — 发现 1 个问题

#### 2c. 密钥扫描
- src/config.ts:12 — 疑似 OpenAI API Key: sk-ab********************xy

### 3. README 检查
[警告] — 缺少中文版本

---
总结：1 项通过，1 项警告，1 项失败
```

## 扫描内容

### 密钥模式

| 模式 | 示例 |
|------|------|
| AWS Access Key | `AKIA...` |
| OpenAI API Key | `sk-...` |
| GitHub PAT | `ghp_...` / `gho_...` |
| GitLab PAT | `glpat-...` |
| Slack Token | `xoxb-...` / `xoxp-...` |
| 私钥 | `-----BEGIN RSA PRIVATE KEY-----` |
| 连接字符串 | `mongodb://user:pass@...` |
| 通用密钥 | `api_key = "..."`、`password = "..."` |

### 按项目类型的 Gitignore 规则

| 项目类型 | 识别方式 | 预期规则 |
|---------|---------|---------|
| Node.js | `package.json` | `node_modules/`、`*.log`、`.env` |
| Python | `pyproject.toml`、`requirements.txt` | `__pycache__/`、`*.pyc`、`.venv/`、`*.egg-info/` |
| Rust | `Cargo.toml` | `target/` |
| Go | `go.mod` | 二进制产物 |
| Java/Kotlin | `pom.xml`、`build.gradle` | `target/`、`build/`、`*.class` |

## 文件结构

```
pre-push-check/
├── SKILL.md                    # Claude Code 技能定义
├── references/
│   └── readme-template.md      # README 生成模板
├── README.md                   # 英文版
├── README-zh_CN.md             # 中文版（本文件）
└── LICENSE                     # MIT 许可证
```

## 贡献

欢迎贡献！请在 [GitHub](https://github.com/royfhs/pre-push-check) 上提交 Issue 或 Pull Request。

## 许可证

MIT 许可证。详见 [LICENSE](LICENSE)。
