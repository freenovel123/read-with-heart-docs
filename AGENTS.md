# AGENTS.md - Read With Heart Docs

## 项目概述
使用 MkDocs 和 Material for MkDocs 构建的 Read With Heart / ParsingBook 文档站，主要维护书源制作、JS API、正文富文本、自定义 TTS 和 Web 写源接口教程。

## 常用命令

### 本地预览
```bash
mkdocs serve
```

### 构建检查
```bash
mkdocs build
```

## 文档规范
- 默认使用中文编写说明。
- 新增 JS API 时，同步说明用途、入参、返回值和最小示例。
- 面向制源用户时优先说明“为什么需要这个能力”和“对写源用户的影响”。
- 保持 `mkdocs.yml` 导航与 `docs/` 下文档入口一致。

## 项目结构
```text
read-with-heart-docs/
├── docs/        # MkDocs 文档内容
├── mkdocs.yml   # 站点配置和导航
└── README.md    # 本地使用说明
```

## 沟通语言
项目中所有注释和文档请使用中文。
