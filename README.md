# claude-feishu

A personal Claude plugin + marketplace that gives Claude access to our company
**Feishu (飞书 / feishu.cn)** workspace — **documents, spreadsheets, and tasks** —
via the official [`lark-cli`](https://github.com/larksuite/cli) running on a
**persistent VPS** reached through the `dawn-vps` MCP.

Built as our own git-hosted plugin so the customization persists across all
Claude sessions (the Anthropic plugin-directory copy of "Lark/Feishu Integration"
is server-hosted and cannot be edited, and targets larksuite.com — not feishu.cn).

## Install

```
/plugin marketplace add hanami-ume/claude-feishu
/plugin install feishu@claude-feishu
```

Then remove the old catalog plugin ("Lark/Feishu Integration") to avoid conflicts.

## What you can ask

| Capability | Example |
|------------|---------|
| **Documents** (CRUD) | "总结这篇飞书文档", "新建一篇 Q4 计划文档", "在知识库里搜索入职指南" |
| **Sheets** (CRUD) | "读一下这个表格的数据", "把这些结果写到 B 列", "新建一个带表头的表格" |
| **Tasks** | "我有哪些待办?", "新建一个任务", "把这个任务标记完成" |

Scoped to docs / sheets / tasks (+ drive, wiki). Other `lark-cli` domains
(calendar, messages, bitable, mail, minutes…) can be enabled by re-running
`auth login` with more `--domain`s — see `/feishu:lark-setup`.

## How it works

- Tool: official `@larksuite/cli` (binary `lark-cli`), `--brand feishu`, App ID `cli_aaff2640f5399cc8`.
- Runtime: runs on the persistent **dawn-vps** workspace (not the ephemeral session
  container). Config + OAuth tokens live there and are reused across sessions.
- The `lark-feishu` skill routes every Feishu request to the VPS via
  `mcp__dawn-vps__shell_exec`. First-time/re-auth setup: `/feishu:lark-setup`.

## Contents

```
.claude-plugin/marketplace.json     # marketplace manifest
feishu/
  .claude-plugin/plugin.json        # plugin manifest
  commands/lark-setup.md            # /feishu:lark-setup
  skills/lark-feishu/SKILL.md       # auto-invoked Feishu access skill
```

## Security notes

- App Secret is provided to `lark-cli` via stdin and stored encrypted on the VPS —
  never committed to this repo.
- OAuth is user-identity device-flow; tokens (refresh ~7 days) live on the VPS.
