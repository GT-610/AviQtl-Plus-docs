---
title: Effects and objects
description: Build native JSON and WGSL render packages for AviQtl-Plus.
---

# Effects and objects

The Slint edition loads third-party Effects and Objects through the native `aviqtl-wgsl-v1` runtime. QML is not a package runtime and legacy QML packages are rejected during installation.

The existing package manager still handles repository metadata, downloads, SHA-256 verification, safe ZIP extraction, atomic upgrades, rollback, and removal. The native runtime defines only the render metadata and shader contract.

## Choose a type

- An **effect** receives the current clip image and returns a processed image.
- An **object** receives a transparent canvas matching the scene dimensions and generates content on it.

Both types use the same WGSL function, parameter model, wgpu render pass, and preview/export path.

## Package structure

```text
my-package/
├── manifest.json
├── README.md
└── my-effect/
    ├── MyEffect.json
    └── MyEffect.wgsl
```

The package ZIP may contain one wrapper directory. Each Effect or Object metadata file can be nested below it, and its shader path is resolved relative to that metadata file. Paths that are absolute, contain traversal components, leave the package through a link, or point to files larger than the runtime limit are rejected.

## Metadata

```json
{
  "id": "my_effect",
  "name": "My Effect",
  "version": "1.0.0",
  "kind": "effect",
  "categories": ["Custom"],
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
        "label": "Amount",
        "min": 0.0,
        "max": 1.0,
        "step": 0.01
      },
      {
        "type": "color",
        "param": "tint",
        "label": "Tint"
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

Required rules:

- `id` and `name` must be non-empty.
- `version` uses three numeric components such as `1.0.0`.
- `kind` is `effect` or `object` and must match the package repository type.
- `categories` contains at least one string.
- `ui.controls` is present; the application builds standard Slint controls from it.
- `runtime.engine` is exactly `aviqtl-wgsl-v1`.
- `runtime.shader` is a safe relative `.wgsl` path.
- `runtime.uniforms` contains at most 16 unique metadata parameter names.

The standard control types include numbers (`float`, `number`, `slider`, `spinner`), integers (`int`, `integer`), booleans, colors, text, files, and options. Parameters use the same editing and keyframe paths as built-in effects.

## Shader contract

The package provides this function:

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

The host supplies:

- `aviqtl_parameter(index) -> vec4<f32>` in the order listed by `runtime.uniforms`;
- `aviqtl_sample(uv) -> vec4<f32>` for sampling the unmodified input texture.

Parameter encoding:

| JSON value | WGSL value |
| --- | --- |
| Number | `.x` |
| Boolean | `.x`, where true is `1.0` |
| Color | `.rgba`, normalized to 0–1 |
| Numeric array | Up to four vector components |

Package source may define helper functions but cannot declare `@group`, `@binding`, `@vertex`, `@fragment`, or `@compute`. The host owns all GPU resources and entry points. The complete combined shader is parsed and validated before installation.

Colors returned by `aviqtl_effect` are straight-alpha RGBA values. The compositor applies object opacity, transforms, masks, blend modes, and premultiplication afterward.

## Example

The repository contains two validated references:

- [`color-effects`](https://github.com/GT-610/AviQtl-Plus/tree/main/effect-packages/color-effects) demonstrates a native Effect.
- [`weather-objects`](https://github.com/GT-610/AviQtl-Plus/tree/main/effect-packages/weather-objects) demonstrates native rain and snow Objects.

## Test checklist

- Install, update, remove, and reinstall the package through a repository ZIP.
- Confirm every entry appears under the correct Effect or Object catalog.
- Exercise every control and keyframe it at multiple timeline positions.
- Test multiple scene sizes and non-default frame rates.
- Compare preview with an exported still or video frame.
- Test missing, invalid, and oversized shader files before publishing.
