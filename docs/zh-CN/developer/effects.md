---
title: 效果与对象
description: 为 AviQtl-Plus 开发原生 JSON 与 WGSL 渲染包。
---

# 效果与对象

Slint 版本通过原生 `aviqtl-wgsl-v1` 运行时加载第三方效果和对象。QML 不再作为包运行时，安装时会拒绝旧 QML 包。

现有包管理器继续负责仓库元数据、下载、SHA-256 校验、安全 ZIP 解压、原子升级、回滚和卸载。原生运行时只定义渲染元数据与着色器契约。

## 选择类型

- **效果（effect）**接收当前片段图像并返回处理后的图像。
- **对象（object）**接收一张与场景尺寸相同的透明画布并在其上生成内容。

两种类型使用相同的 WGSL 函数、参数模型、wgpu 渲染 pass，以及相同的预览和导出路径。

## 包结构

```text
my-package/
├── manifest.json
├── README.md
└── my-effect/
    ├── MyEffect.json
    └── MyEffect.wgsl
```

包 ZIP 可以包含一层包装目录。效果或对象的元数据文件可以位于其任意子目录，着色器路径相对于对应元数据文件解析。绝对路径、含路径穿越的路径、通过链接离开包目录的文件，以及超过运行时大小限制的文件都会被拒绝。

## 元数据

```json
{
  "id": "my_effect",
  "name": "我的效果",
  "version": "1.0.0",
  "kind": "effect",
  "categories": ["自定义"],
  "params": {
    "amount": 0.5,
    "enabled": true,
    "tint": "#ffffffff"
  },
  "ui": {
    "controls": [
      {
        "type": "slider",
        "param": "amount",
        "label": "强度",
        "min": 0.0,
        "max": 1.0,
        "step": 0.01
      },
      {
        "type": "color",
        "param": "tint",
        "label": "颜色"
      }
    ]
  },
  "runtime": {
    "engine": "aviqtl-wgsl-v1",
    "shader": "MyEffect.wgsl",
    "uniforms": ["amount", "enabled", "tint"]
  }
}
```

必要规则：

- `id` 和 `name` 不能为空。
- `version` 使用 `1.0.0` 这样的三段数字版本号。
- `kind` 只能为 `effect` 或 `object`，并且必须与仓库中的包类型一致。
- `categories` 至少包含一个字符串。
- 必须包含 `ui.controls`；程序会据此生成统一的 Slint 控件。
- `runtime.engine` 必须为 `aviqtl-wgsl-v1`。
- `runtime.shader` 必须是安全的相对 `.wgsl` 路径。
- `runtime.uniforms` 最多包含 16 个不重复的元数据参数名。

标准控件类型包括数值（`float`、`number`、`slider`、`spinner`）、整数（`int`、`integer`）、布尔值、颜色、文本、文件和选项。包参数与内置效果共用编辑和关键帧路径。

## 着色器契约

包提供以下函数：

```wgsl
fn aviqtl_effect(
    input_color: vec4<f32>,
    uv: vec2<f32>,
    canvas_size: vec2<f32>,
    time_seconds: f32,
) -> vec4<f32> {
    let amount = aviqtl_parameter(0u).x;
    let enabled = aviqtl_parameter(1u).x > 0.5;
    let tint = aviqtl_parameter(2u);
    if enabled {
        return mix(input_color, vec4<f32>(tint.rgb, input_color.a), amount);
    }
    return input_color;
}
```

宿主提供：

- `aviqtl_parameter(index) -> vec4<f32>`，顺序与 `runtime.uniforms` 一致；
- `aviqtl_sample(uv) -> vec4<f32>`，用于采样未经当前效果修改的输入纹理。

参数编码：

| JSON 值 | WGSL 值 |
| --- | --- |
| 数字 | `.x` |
| 布尔值 | `.x`，true 为 `1.0` |
| 颜色 | `.rgba`，归一化至 0–1 |
| 数字数组 | 最多填充四个向量分量 |

包源码可以定义辅助函数，但不能声明 `@group`、`@binding`、`@vertex`、`@fragment` 或 `@compute`。所有 GPU 资源与入口点均由宿主管理。安装前会解析并验证合成后的完整着色器。

`aviqtl_effect` 返回直通 Alpha 的 RGBA 值。之后由合成器应用对象不透明度、变换、遮罩、混合模式和预乘处理。

## 示例

仓库提供两个通过验证的参考包：

- [`color-effects`](https://github.com/GT-610/AviQtl-Plus/tree/main/effect-packages/color-effects) 演示原生效果。
- [`weather-objects`](https://github.com/GT-610/AviQtl-Plus/tree/main/effect-packages/weather-objects) 演示原生雨雪对象。

## 测试清单

- 通过仓库 ZIP 安装、升级、卸载并重新安装包。
- 确认每个条目出现在正确的效果或对象目录。
- 操作全部控件，并在多个时间位置设置关键帧。
- 测试多种场景尺寸和非默认帧率。
- 对比预览与导出的静帧或视频帧。
- 发布前测试缺失、无效和过大的着色器文件。
