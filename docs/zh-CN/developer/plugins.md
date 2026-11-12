---
title: 插件开发
description: 创建用于自动化 Freya NLE 工作流的 Lua 插件。
---

# 插件开发

Lua 插件可以自动化播放控制、时间线、项目、场景、设置等工作流。插件受权限控制，并可以向用户公开声明式参数。

## 推荐结构

```text
my_plugin/
├── manifest.lua
├── main.lua
└── README.md
```

`manifest.lua` 标识插件及其最低应用版本；`main.lua` 包含生命周期钩子和 API 调用。

```lua
return {
    id = "com.example.my-plugin",
    name = "My Plugin",
    version = "0.1.0",
    author = "Example Author",
    description = "A small Freya NLE automation plugin",
    min_app_version = "0.2.0"
}
```

## 安全模型

新插件不会自动获得敏感权限。只申请实际需要的能力，正确处理权限错误，并让文件或网络访问对用户保持可见。

## 开发原则

- 保持高频更新钩子轻量。
- 使用插件 ID 作为持久设置命名空间。
- 使用反向域名 ID 避免冲突。
- 正确处理错误，避免一个插件中断编辑会话。
- 记录安装、权限和移除方法。

主仓库在 [`plugins/`](https://github.com/GT-610/Freya-NLE/tree/main/plugins) 和 [`examples/`](https://github.com/GT-610/Freya-NLE/tree/main/examples) 中提供示例。脚本接口稳定后，本区域会加入完整 API 参考。
