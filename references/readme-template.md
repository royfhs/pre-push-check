# README 模板参考

当需要为用户生成或补全 README 时，按照以下模板结构。根据项目实际情况填写内容，没有的部分可以省略，但核心章节必须保留。

## 模板结构

```markdown
<div align="center">

<img src="{{LOGO_URL}}" alt="Logo" width="80">

####

<!-- 主要徽章行（大尺寸）：官网、文档、社区 -->
[![Website](https://img.shields.io/badge/Official%20Website-{{SITE}}-teal?style=for-the-badge&logo=world&logoColor=white&color=0891b2)]({{WEBSITE_URL}})
[![Documentation](https://img.shields.io/badge/Documentation-DOCS-f472b6?logo=googledocs&logoColor=white&style=for-the-badge)]({{DOCS_URL}})

<!-- 次要徽章行（小尺寸）：版本号、发布、Docker、社交 -->
![GitHub Release](https://img.shields.io/github/v/release/{{OWNER}}/{{REPO}}?style=flat&logo=github)

<!-- 多语言链接 -->
[English](README.md) | [中文](README-zh_CN.md)

</div>

# {{EMOJI}} {{项目名称}}

**{{项目名称}}是{{一句话描述}}。**

{{2-3 句话展开说明核心功能和技术特点。}}

## 为什么选择 {{项目名称}}？

- {{痛点 1}}
- {{痛点 2}}
- {{本项目的解决方案}}

## 架构

{{架构描述}}

<div align="center">
<img align="center" height="500" src="{{架构图链接}}">
</div>

## 演示

{{演示视频/GIF/截图}}

## 特性

- 📝 {{特性 1}}
- 🌐 {{特性 2}}
- 🖥️ {{特性 3}}

## 📖 文档

请参阅[完整文档]({{文档链接}})。

## 快速开始

> **步骤 1** — 下载项目
> **步骤 2** — 安装依赖
> **步骤 3** — 配置环境变量
> **步骤 4** — 启动服务

## 🚀 贡献

## ✉️ 联系我们

## 🛡 免责声明

---

<!-- Star History 图表 -->
```

## 核心章节（必须存在）

以下章节是 README 质量检查的最低要求：

1. **项目标题 + 一句话描述** — 必须有
2. **特性/功能列表** — 必须有，至少 3 项
3. **快速开始/安装步骤** — 必须有，包含具体的安装和运行命令
4. **双语内容** — 必须同时有中文和英文（可以分文件或在同一文件中）

## 推荐章节（建议存在）

5. **徽章区域** — 版本号、CI 状态、许可证等
6. **架构/原理说明** — 含架构图更佳
7. **演示** — 视频、GIF 或截图
8. **文档链接** — 指向详细文档
9. **贡献指南** — 或链接到 CONTRIBUTING.md
10. **许可证/免责声明**

## 多语言组织方式

推荐采用分文件方式：
- `README.md` — 英文版（默认）
- `README-zh_CN.md` — 中文版
- 在文件顶部用语言链接互相引用

也可以在同一文件中用分隔线隔开两种语言，但分文件方式更清晰。
