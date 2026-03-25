---
name: pre-push-check
description: "Two modes: (1) Pre-push check — scans .gitignore, sensitive data, README compliance before pushing to GitHub. Trigger on: push, ship, 推送, 上传代码, 提交到 GitHub, create PR, or check repo for secrets/gitignore/README. (2) README generator — generates a professional bilingual README from a template. Trigger on: 写 README, 生成 README, write README, create README, help me write a README, 帮我写个 README. Proactively suggest when the user is preparing to push code OR when they need a README."
---

# Pre-Push Repository Check

你是一个推送前的质量检查门禁。在代码推送到 GitHub 之前，系统性地检查三个方面：gitignore 配置、敏感数据暴露、README 双语合规。目标是在代码公开之前拦截危险或不规范的内容。

**所有输出必须使用中文。** 报告标题、描述、详情、修复建议——全部用中文。技术术语（如文件名、命令、路径）保持原样即可。

## 两种模式

本 skill 有两种运行模式，根据用户意图自动判断：

### 模式 A：推送前检查
当用户提到 push、推送、ship、创建 PR 等操作时触发。执行完整的三项检查（gitignore、敏感数据、README），生成报告并提供修复。

### 模式 B：README 生成
当用户要求写 README、生成 README 时触发。仅执行 README 生成流程，不做其他检查。具体流程见下方「README 生成流程」章节。

## 触发时机

- 用户手动调用 `/pre-push-check`
- 用户即将 push、ship 或创建 PR 时——进入模式 A
- 用户要求写/生成 README 时——进入模式 B

---

# 模式 A：推送前检查

## 检查流程

运行全部三项检查，然后呈现统一报告。不要在第一个失败项就停下来——用户需要看到完整的结果。

### 1. Gitignore 审查

目标：确保不应被版本控制追踪的文件已被正确忽略。

**第一步** — 读取现有的 `.gitignore`（如果存在的话）。

**第二步** — 扫描仓库中被 git 追踪但本应被忽略的文件。使用 `git ls-files` 查看当前被追踪的文件列表，标记属于以下类别的文件：

- **构建产物**：`node_modules/`、`dist/`、`build/`、`__pycache__/`、`*.pyc`、`.next/`、`target/`、`bin/`、`obj/`、`*.class`、`*.o`、`*.so`、`*.dll`、`*.exe`
- **IDE/编辑器文件**：`.idea/`、`.vscode/`（除非是团队共享配置）、`*.swp`、`*.swo`、`.DS_Store`、`Thumbs.db`、`*.suo`、`*.user`
- **依赖目录**：`node_modules/`、`vendor/`（视情况而定）、`venv/`、`.venv/`、`env/`、`.env/`（指目录，非文件）
- **系统文件**：`.DS_Store`、`Thumbs.db`、`desktop.ini`、`._*`
- **日志文件**：`*.log`、`logs/`、`npm-debug.log*`、`yarn-debug.log*`、`yarn-error.log*`

**第三步** — 检查 `.gitignore` 是否缺少当前项目类型常用的忽略规则：
- Node.js 项目（有 `package.json`）→ 应忽略 `node_modules/`、`*.log`、`.env`
- Python 项目（有 `setup.py`/`pyproject.toml`/`requirements.txt`）→ 应忽略 `__pycache__/`、`*.pyc`、`.venv/`、`*.egg-info/`
- Java/Kotlin（有 `pom.xml`/`build.gradle`）→ 应忽略 `target/`、`build/`、`*.class`
- Go（有 `go.mod`）→ 视情况而定，通常忽略二进制产物
- Rust（有 `Cargo.toml`）→ 应忽略 `target/`

### 2. 敏感数据与配置文件检查

目标：防止密钥和敏感配置被推送到远程仓库。

**第 2a 步 — 文档、日志与配置文件检查**

扫描被追踪的文件（`git ls-files`），查找：
- 可能属于内部文档的内容：`doc/`、`docs/` 中带有内部/私密标记的文件
- 日志文件：`*.log`、`logs/`
- 常含敏感信息的配置文件：`config.yml`、`config.json`、`application.properties`、`appsettings.json`、`database.yml`、`credentials.*`

对于配置文件，需读取内容判断是否包含真实值（而非占位符）。`config.yml` 中 `password: changeme` 是可以接受的；`password: aR3alP@ssw0rd!` 则不行。

**第 2b 步 — .env 文件检查**

- 检查是否有 `.env` 文件（`.env`、`.env.local`、`.env.production`、`.env.development` 等）被 git 追踪（`git ls-files`）。
- 如果 `.env` 被追踪，标记为严重问题。
- 检查是否存在 `.env.sample` 或 `.env.example`。如果不存在，建议基于 `.env` 创建一个，用占位符替换真实值。
- 验证 `.gitignore` 中是否包含 `.env*` 模式（同时允许 `.env.sample` 和 `.env.example` 通过）。

**第 2c 步 — 密钥与 API Key 扫描**

搜索被追踪文件中疑似密钥的模式。使用 `git ls-files` 获取文件列表，然后 grep 搜索：

- **API 密钥**：匹配 `AKIA[0-9A-Z]{16}`（AWS）、`sk-[a-zA-Z0-9]{20,}`（OpenAI 风格）、`ghp_[a-zA-Z0-9]{36}`（GitHub PAT）、`gho_`、`glpat-`、`xox[bpsa]-` 等模式
- **通用密钥**：匹配 `api_key\s*[:=]\s*['"][^'"]+['"]`、`secret\s*[:=]\s*['"][^'"]+['"]`、`password\s*[:=]\s*['"][^'"]+['"]`、`token\s*[:=]\s*['"][^'"]+['"]` 等模式——但跳过值明显是占位符的行（空值、`xxx`、`changeme`、`your-key-here`、`TODO`、`<.*>`、`REPLACE_ME` 等）
- **私钥文件**：`-----BEGIN (RSA |EC |DSA |OPENSSH )?PRIVATE KEY-----`
- **连接字符串**：`mongodb(\+srv)?://[^/\s]+`、`postgres(ql)?://[^/\s]+`、`mysql://[^/\s]+`、`redis://[^/\s]+` ——如果包含凭据（user:pass@）则标记
- **可疑上下文中的 Base64 长字符串**（赋值给名为 `key`、`secret`、`token`、`credential` 的变量）

跳过二进制文件、锁文件（`package-lock.json`、`yarn.lock`、`Cargo.lock` 等）以及 vendor/生成的代码。

发现疑似密钥时，显示文件名、行号和匹配的模式——但对实际值进行脱敏处理（仅显示前 4 位和后 2 位，其余用 `*` 遮盖）。

### 3. README 检查

目标：确保 README.md 同时包含中文和英文内容，并且结构完整。

模板参考文件在 `references/readme-template.md`，检查时读取该文件了解标准结构。

#### 3a. 双语检查

- 读取 `README.md`（如果不存在则标记为缺失）。
- 检查是否包含中文字符（Unicode 范围 `\u4e00-\u9fff`）。
- 检查是否包含实质性的英文内容（不仅仅是代码片段中的英文）。
- 如果缺少任一语言，标记并建议需要补充的内容。
- 检查多语言组织方式：是否有 `README-zh_CN.md` 等分文件，或在同一文件中用分隔线隔开。

#### 3b. 结构完整性检查

对照模板检查 README 是否包含以下**核心章节**（缺失任何一项标记为警告）：

1. **项目标题 + 一句话描述** — 开头必须有项目名和简要描述
2. **特性/功能列表** — 必须有功能列表，至少 3 项
3. **快速开始/安装步骤** — 必须包含具体的安装和运行命令
4. **双语内容** — 必须同时有中文和英文

同时检查以下**推荐章节**（缺失时标记为提示，不影响通过/失败）：

5. **徽章区域** — 版本号、CI 状态、许可证等 shields.io 徽章
6. **架构/原理说明** — 系统架构描述，最好含架构图
7. **演示** — 视频、GIF 或截图
8. **文档链接** — 指向详细文档
9. **贡献指南** — 或链接到 CONTRIBUTING.md
10. **许可证/免责声明**

## 报告格式

完成所有检查后，以中文呈现结构化报告：

```
## 推送前检查报告

### 1. Gitignore 配置
[通过/警告/失败] — 概要
- 详情 1
- 详情 2

### 2. 敏感数据检查
[通过/警告/失败] — 概要

#### 2a. 文档/日志/配置文件
- 详情

#### 2b. .env 文件
- 详情

#### 2c. 密钥扫描
- 详情

### 3. README 检查
[通过/警告/失败] — 概要

#### 3a. 双语检查
- 详情

#### 3b. 结构完整性
- 核心章节：X/4 项齐全
- 推荐章节：X/6 项存在
- 缺失项列表

---
总结：X 项通过，Y 项警告，Z 项失败
```

严重级别：
- **通过**：未发现问题
- **警告**：轻微问题，建议处理但不影响推送（例如缺少 `.env.sample`、gitignore 可以更完善）
- **失败**：严重问题，推送前必须修复（例如被追踪的 `.env` 包含真实密钥、代码中有 API Key、README 缺少某种语言）

## 修复流程

报告呈现后，如果有任何警告或失败项：

1. 列出所有可修复的问题并编号
2. 询问用户："以上哪些问题需要我帮你修复？（输入编号、'全部' 或 '跳过'）"
3. 对用户选择的每个修复项：
   - 说明计划做的改动
   - 执行修改
   - 确认完成
4. 修复完成后，重新运行相关检查以验证

常见的自动修复操作：
- 向 `.gitignore` 添加缺失的规则
- 将被追踪的 `.env` 从 git 中移除（`git rm --cached`）
- 基于 `.env` 生成 `.env.sample`，用占位符替换真实值
- 为 README.md 添加缺失的语言版本——读取 `references/readme-template.md` 中的模板，基于项目信息生成完整的 README 框架（含徽章、特性列表、快速开始等章节），供用户填写细节
- 补全 README 缺失的章节——对照模板检查结果，为缺失的核心章节生成骨架内容插入到现有 README 中

对于代码中发现的硬编码密钥，不要自动修复——仅标记。用户需要自行决定如何处理已泄露凭据的轮换。

---

# 模式 B：README 生成

当用户要求写 README 或生成 README 时进入此模式。不执行 gitignore 和敏感数据检查，仅生成 README。

## 生成流程

**第一步 — 了解项目**

扫描当前仓库，收集以下信息：
- 项目类型（通过 `package.json`、`pyproject.toml`、`go.mod`、`Cargo.toml` 等判断）
- 项目名称和描述（从配置文件或目录名推断）
- 主要依赖和技术栈
- 入口文件和启动方式
- 是否有已有的 README（如果有，读取现有内容作为参考）

**第二步 — 确认关键信息**

向用户确认以下内容（如果无法从代码中推断）：
- 项目的一句话描述
- 核心功能/特性列表
- 是否有线上地址、文档链接、Discord/社区链接
- GitHub 仓库的 owner/repo（用于生成徽章）
- 其他需要特别说明的内容

**第三步 — 生成 README**

读取 `references/readme-template.md` 中的模板，按以下规则生成：

1. **生成两个文件**：`README.md`（英文）和 `README-zh_CN.md`（中文），顶部互相链接
2. **必须包含的章节**：
   - 居中 Logo + 徽章区域（根据项目类型选择合适的徽章）
   - 多语言链接
   - 项目标题 + 一句话描述
   - 特性列表（至少 3 项，带 emoji）
   - 快速开始（具体的安装和运行命令，从项目配置中提取）
3. **按需包含的章节**（项目有相关内容时才加）：
   - "为什么选择本项目"（如果用户提供了背景）
   - 架构说明（如果项目较复杂）
   - 演示（占位，提示用户补充视频/截图）
   - 文档链接
   - 贡献指南
   - 许可证/免责声明
   - Star History 图表

**第四步 — 展示并确认**

生成后展示 README 内容，询问用户是否满意，是否需要调整某些章节的内容或增减章节。根据反馈修改后写入文件。
