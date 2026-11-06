---
name: lark-feishu
description: Access our company Feishu (飞书 / feishu.cn) — documents, spreadsheets, and tasks — via lark-cli running on a persistent VPS (dawn-vps). Use when the user mentions 飞书 / Feishu / Lark, a feishu.cn doc/sheet/wiki/task URL, or asks to read/create/edit a Feishu 文档/表格/任务 (docs, sheets, tasklists).
---

# 飞书 (Feishu) access — routed to the persistent VPS

Our company's Feishu integration runs the **official `lark-cli`** on a **persistent
dawn-vps workspace**, NOT in the local session container. Config and OAuth tokens
live on the VPS and survive across sessions, so you do not re-install or re-auth
every session.

- Brand/region: **feishu** (`open.feishu.cn`)
- App ID: `cli_aaff2640f5399cc8`
- Authorized scopes: `docs, sheets, task, drive, wiki` (full CRUD on docs/sheets, tasks read+write)

## How to run lark-cli (ALWAYS via dawn-vps)

Execute every `lark-cli` command through the **`mcp__dawn-vps__shell_exec`** tool,
prefixing the PATH each call (each shell is fresh). Do NOT run `lark-cli` in the
local container — it is not installed there.

```
mcp__dawn-vps__shell_exec(command:
  'export PATH="$HOME/.local/node/bin:$HOME/.npm-global/bin:$PATH"; lark-cli <ARGS>')
```

If `mcp__dawn-vps__shell_exec` is not in your toolset, load it first with
ToolSearch (`select:mcp__dawn-vps__shell_exec`). Call `mcp__dawn-vps__auth_identity`
once to confirm enrollment.

## Step 0 — health check (run first each session)

```bash
export PATH="$HOME/.local/node/bin:$HOME/.npm-global/bin:$PATH"
lark-cli auth status
```

- `identities.user.status: ready` → good, proceed.
- user missing / token expired → re-auth with the device flow (see bottom).

## Common commands

Prefer `--json` for machine-readable output. Discover subcommands with
`lark-cli <group> -h`.

### Documents (docx) — CRUD
```bash
lark-cli docs +search --query "<keywords>" --json       # find docs/wiki/sheets
lark-cli docs +fetch --url "<doc url or token>" --json   # read content (markdown/blocks)
lark-cli docs +create --title "<title>" --json           # create
lark-cli docs +update ... --json                         # edit (see docs +update -h)
```

### Spreadsheets (sheets) — CRUD
```bash
lark-cli sheets +workbook-create --title "<t>" --values '[["A","B"],["1","2"]]' --json
lark-cli sheets +workbook-info --url "<sheet url>" --json
lark-cli sheets +cells-get --url "<sheet url>" --range "Sheet1!A1:B10" --json
lark-cli sheets +table-put ...   --json   # write typed table (see -h)
```

### Tasks (todo) — read + write
```bash
lark-cli task +get-my-tasks --json
lark-cli task +create --summary "<text>" --json
lark-cli task +complete --guid "<task guid>" --json
```

For exact flags of any command, run it with `-h` on the VPS, or consult the
official `lark-doc` / `lark-sheets` / `lark-task` skill docs via
`lark-cli skills read <lark-doc|lark-sheets|lark-task>`.

## Re-authorization (only if `auth status` shows the user token expired)

The refresh token lasts ~7 days; after that, re-run the device flow:

```bash
export PATH="$HOME/.local/node/bin:$HOME/.npm-global/bin:$PATH"
lark-cli auth login --domain docs,sheets,task,drive,wiki --no-wait --json
# take verification_url + device_code from the output, then:
lark-cli auth qrcode --output "$HOME/lark-auth-qr.png" "<verification_url>"
```

Pull the QR PNG back (`base64 -w0 $HOME/lark-auth-qr.png`), show it to the user
with the URL + user_code, and after they authorize as themselves, complete it:

```bash
lark-cli auth login --device-code "<device_code>" --json
```

## First-time VPS provisioning (only if lark-cli is missing on the VPS)

See `/feishu:lark-setup`. In short, on the dawn-vps:
1. install Node locally (`~/.local/node`) and set PATH in `~/.bashrc`
2. `npx -y @larksuite/cli@latest install`
3. `printf '%s' '<APP_SECRET>' | lark-cli config init --app-id cli_aaff2640f5399cc8 --brand feishu --app-secret-stdin --lang zh_cn`
4. device-flow `auth login` as above
