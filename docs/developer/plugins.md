---
title: Plugin development
description: Create Lua plugins that automate Freya NLE workflows.
---

# Plugin development

Lua plugins can automate transport, timeline, project, scene, settings, and
other application workflows. Plugins are permission-controlled and can expose
declarative parameters to users.

## Recommended structure

```text
my_plugin/
├── manifest.lua
├── main.lua
└── README.md
```

`manifest.lua` identifies the plugin and its minimum application version.
`main.lua` contains lifecycle hooks and API calls.

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

## Security model

New plugins do not automatically receive sensitive permissions. Request only
the capabilities the plugin needs, handle permission errors, and keep file or
network access visible to users.

## Development principles

- Keep frequently called update hooks lightweight.
- Namespace persistent settings with the plugin ID.
- Use reverse-domain IDs to avoid collisions.
- Handle errors so one plugin does not interrupt an editing session.
- Document installation, permissions, and removal.

The main repository contains example plugins under
[`plugins/`](https://github.com/GT-610/Freya-NLE/tree/main/plugins) and
[`examples/`](https://github.com/GT-610/Freya-NLE/tree/main/examples). A full
API reference will be added here as the scripting surface stabilizes.
