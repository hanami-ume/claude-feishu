---
description: Set up / troubleshoot our company's Feishu (飞书) access — official lark-cli on the persistent dawn-vps
---

Set up or repair our company Feishu integration. **lark-cli runs on the persistent
dawn-vps**, not in the local session container — config and OAuth tokens live there
and survive across sessions.

> Company profile:
> - Brand/region: **feishu** (国内飞书, `open.feishu.cn`) — NOT larksuite.
> - App ID: `cli_aaff2640f5399cc8`
> - Scopes: **docs, sheets, task, drive, wiki** (docs/sheets CRUD, tasks read+write)
> - Tool: official `@larksuite/cli` (binary `lark-cli`). Do NOT use the third-party
>   `seahyc/lark-cli` — it hardcodes larksuite.com and cannot reach feishu.cn.

All commands below run **on the VPS** via `mcp__dawn-vps__shell_exec`
(load it with ToolSearch `select:mcp__dawn-vps__shell_exec` if needed). Prefix PATH
every call: `export PATH="$HOME/.local/node/bin:$HOME/.npm-global/bin:$PATH"`.

Why the VPS and not the local container: this is a cloud session whose container is
ephemeral (install + tokens are wiped at session end) and whose outbound SSH (port 22)
is blocked. The dawn-vps MCP is account-level and persistent, so it is the durable home.

## 1. Confirm enrollment + health

```bash
mcp__dawn-vps__auth_identity            # confirm enrolled
# then via shell_exec:
export PATH="$HOME/.local/node/bin:$HOME/.npm-global/bin:$PATH"
lark-cli --version && lark-cli config show && lark-cli auth status
```

- `auth status` → `identities.user.status: ready` means you're done; just use the
  `lark-feishu` skill.
- `lark-cli` missing → provision (step 2). user token expired → re-auth (step 3).

## 2. Provision lark-cli on the VPS (first time only)

```bash
# Node (user-local), persisted on PATH
mkdir -p "$HOME/.local" && cd "$HOME/.local" \
  && curl -fsSL -o node.tar.xz https://nodejs.org/dist/v22.11.0/node-v22.11.0-linux-x64.tar.xz \
  && tar -xf node.tar.xz && mv node-v22.11.0-linux-x64 node && rm -f node.tar.xz
grep -q '/.local/node/bin' "$HOME/.bashrc" || \
  echo 'export PATH="$HOME/.local/node/bin:$HOME/.npm-global/bin:$PATH"' >> "$HOME/.bashrc"
export PATH="$HOME/.local/node/bin:$HOME/.npm-global/bin:$PATH"
npm config set prefix "$HOME/.npm-global"

# official CLI + skills
npx -y @larksuite/cli@latest install

# config (App Secret via stdin — ask the user to paste it; never echo/commit it)
printf '%s' '<APP_SECRET>' | lark-cli config init \
  --app-id cli_aaff2640f5399cc8 --brand feishu --app-secret-stdin --lang zh_cn
lark-cli auth scopes      # validates the secret + lists enabled scopes
```

## 3. Authenticate (device-flow — headless friendly)

```bash
lark-cli auth login --domain docs,sheets,task,drive,wiki --no-wait --json
```

1. From the JSON take `verification_url` + `device_code`.
2. `lark-cli auth qrcode --output "$HOME/lark-auth-qr.png" "<verification_url>"`,
   then `base64 -w0 "$HOME/lark-auth-qr.png"` and decode it locally to send the user
   a PNG. Also give them the URL + `user_code`.
3. User opens it, logs in **as themselves** (user identity), approves.
4. Complete: `lark-cli auth login --device-code "<device_code>" --json`
5. Verify: `lark-cli auth status`.

Refresh token lasts ~7 days; within that window no re-scan is needed.

## 4. Smoke test (CRUD)

```bash
lark-cli task +get-my-tasks --json
lark-cli docs +create --title "连接测试" --json
lark-cli sheets +workbook-create --title "表格测试" --values '[["A","B"],["1","2"]]' --json
```

## Day-to-day use

Routine Feishu work does not need this command — the **`lark-feishu`** skill triggers
on Feishu/飞书/doc/sheet/task requests and runs the right `lark-cli` command on the VPS.
