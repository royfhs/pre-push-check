<div align="center">

# 🛡 pre-push-check

**A Claude Code skill that scans your repo for security issues, .gitignore gaps, and README compliance before pushing to GitHub.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

[English](README.md) | [中文](README-zh_CN.md)

</div>

Stop pushing secrets, missing `.gitignore` rules, or incomplete READMEs. `pre-push-check` is a [Claude Code skill](https://code.claude.com/docs/en/skills#extend-claude-with-skills) that acts as a quality gate before your code goes public — it systematically checks three areas and offers to fix what it finds.

## Features

- 🔒 **Secret Scanner** — Detects API keys, private keys, connection strings, and hardcoded passwords with pattern matching (AWS, OpenAI, GitHub PAT, etc.)
- 📁 **Gitignore Auditor** — Verifies `.gitignore` coverage for your project type (Node.js, Python, Rust, Go, Java) and flags tracked files that shouldn't be
- 📝 **README Compliance** — Checks for bilingual content (English + Chinese), required sections, and structural completeness against a template
- 🔧 **Auto-Fix** — Offers to fix issues automatically: add `.gitignore` rules, remove tracked `.env` files, generate missing README sections
- 📄 **README Generator** — Generates professional bilingual READMEs from scratch with badges, feature lists, and quick start guides

## Two Modes

### Mode A: Pre-Push Check

Triggered when you're about to push, ship, or create a PR. Runs all three checks and generates a structured report:

1. **Gitignore audit** — Scans tracked files against known patterns (build artifacts, IDE files, dependencies, system files)
2. **Sensitive data scan** — Checks `.env` files, config files with real values, API keys, private keys, connection strings
3. **README check** — Bilingual content, required sections (title, features, quick start), recommended sections (badges, architecture, contributing)

### Mode B: README Generator

Triggered when you ask to write or generate a README. Scans the project, generates `README.md` (English) + `README-zh_CN.md` (Chinese) with badges, features, and quick start.

## Quick Start

### Install as Claude Code Skill

```bash
# Clone to your skills directory
git clone https://github.com/royfhs/pre-push-check.git ~/.claude/skills/pre-push-check
```

### Usage

In Claude Code, just say:

```
/pre-push-check
```

Or trigger naturally:
- "Check the repo before I push"
- "帮我检查一下能不能推送"
- "Generate a README for this project"
- "帮我写个 README"

## Report Example

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

## What It Scans

### Secret Patterns

| Pattern | Example |
|---------|---------|
| AWS Access Key | `AKIA...` |
| OpenAI API Key | `sk-...` |
| GitHub PAT | `ghp_...` / `gho_...` |
| GitLab PAT | `glpat-...` |
| Slack Token | `xoxb-...` / `xoxp-...` |
| Private Keys | `-----BEGIN RSA PRIVATE KEY-----` |
| Connection Strings | `mongodb://user:pass@...` |
| Generic secrets | `api_key = "..."`, `password = "..."` |

### Gitignore Rules by Project Type

| Project Type | Detected By | Expected Rules |
|-------------|-------------|----------------|
| Node.js | `package.json` | `node_modules/`, `*.log`, `.env` |
| Python | `pyproject.toml`, `requirements.txt` | `__pycache__/`, `*.pyc`, `.venv/`, `*.egg-info/` |
| Rust | `Cargo.toml` | `target/` |
| Go | `go.mod` | Binary artifacts |
| Java/Kotlin | `pom.xml`, `build.gradle` | `target/`, `build/`, `*.class` |

## File Structure

```
pre-push-check/
├── SKILL.md                    # Claude Code skill definition
├── references/
│   └── readme-template.md      # README template for generation
├── README.md                   # This file
├── README-zh_CN.md             # Chinese version
└── LICENSE                     # MIT License
```

## Contributing

Contributions are welcome! Please open an issue or submit a pull request on [GitHub](https://github.com/royfhs/pre-push-check).

## License

MIT License. See [LICENSE](LICENSE) for details.
