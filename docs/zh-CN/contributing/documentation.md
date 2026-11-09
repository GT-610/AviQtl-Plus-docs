---
title: 文档贡献指南
description: 编写、翻译、审阅 Freya NLE 手册并补充截图。
---

# 文档贡献指南

文档维护在独立的 [Aviqtl-Plus-docs 仓库](https://github.com/GT-610/Aviqtl-Plus-docs)。除非你明确要修改文档仓库本身，否则不要通过应用仓库的 submodule 目录提交文档修改。

## 信息架构

- `guide/`：面向任务的用户文档，也是手册主体。
- `reference/`：术语和详细查询资料。
- `developer/`：构建、架构、API、效果与插件。
- `contributing/`：文档流程和项目参与方式。
- `zh-CN/`：与英文目录对应的简体中文页面。
- `public/images/`：截图和其他静态资源。

## 写作风格

- 从读者希望完成的结果开始。
- 工作流使用编号步骤，选择与检查项使用列表。
- 使用准确界面标签，并解释其作用。
- 不要在用户任务中混入开发实现细节。
- 功能或页面不完整时明确说明。
- 不要复制其他视频编辑器手册的文字或图片。

## 英文与简体中文

英文是源语言。先新增或更新英文页面，再更新 `zh-CN/` 下的相同路径。翻译可以稍后补充，但不应加入英文源文档中不存在的功能描述。

## 添加截图

先在 `public/` 下添加图片文件，再使用 VitePress 标准的 Markdown 图片语法：

```md
![对界面状态有意义的描述](/images/zh-CN/guide/example.webp)
```

在 `public/` 下添加与路径对应的图片；上述示例对应 `public/images/zh-CN/guide/example.webp`。英文截图放在 `images/en/`，简体中文截图放在 `images/zh-CN/`。

## 预览修改

```bash
bun install
bun run docs:dev
```

提交拉取请求前运行：

```bash
bun run docs:build
```

本地搜索索引会在 VitePress 构建时自动生成，不需要提交外部搜索服务配置或索引文件。
