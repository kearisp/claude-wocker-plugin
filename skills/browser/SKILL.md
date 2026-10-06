---
name: browser
description: Drive a browser against a running app via the `ws` (wocker) CLI's built-in browser service. Use whenever you need to open/click/screenshot a page in a project that uses `ws` — no project-specific setup needed, these commands work the same in any `ws`-managed project.
---

# Driving the browser via `ws`

The `ws` CLI ships a browser service with its own puppeteer-core. Don't spin
up a separate Playwright/Puppeteer install to check a page — use this instead.

```bash
# Inline JS: `code` is the body of an async function receiving `page` and `helper`
ws browser:eval "await page.goto('...'); return await page.title();" [service]

# Script file: file must export an async function receiving Puppeteer `page` and `helper`
ws browser:exec <file> [service]

# Screenshot a page (or a single element with --selector); saves to
# /tmp/screenshot-<timestamp>.png unless --out is given
ws browser:screenshot [service] --out shot.png --fullpage
ws browser:screenshot [service] --selector "#header" --out header.png
```

`exec`, `eval`, and `screenshot` all target a tab, and never close it
afterwards:
- **default (no `--tab`/`--new`)** — the currently **active** tab (the one
  in the foreground of the browser window). Use this to pick up a tab
  that's already open and authenticated instead of logging in again.
- `--tab <index|url-substring>` — a specific open tab, by index (see
  `ws browser:pages`, which marks the active one) or a substring of its URL.
- `--new` — always open a fresh tab instead. Mutually exclusive with `--tab`.
- `--viewport WIDTHxHEIGHT` (e.g. `1280x800`) — override the page size;
  default is to match the browser window's own size.

## Project helper: env, meta, secrets

`eval` and `exec` also receive a second argument, `helper`, scoped to the
project you run the command from (same data as `ws config`, `ws meta`,
`ws secret:*`). In `exec` the signature is `async (page, helper) => {...}`;
in `eval` both `page` and `helper` are in scope.

```ts
helper.getEnv(key, byDefault?)  / helper.hasEnv(key)
helper.getMeta(key, byDefault?) / helper.hasMeta(key)
helper.generatePassword(name, length?)   // Promise<void>
helper.setSecret(name, value)            // Promise<void>
helper.hasSecret(name)                   // Promise<boolean>
helper.fillWithSecret(selector, name)    // Promise<void>
```

`getEnv`/`getMeta` return plain values (URLs, flags) — use them for
non-secret config (`ws config:set`, `ws meta:set` to manage).

### Secrets — how to use them

Secrets are for credentials that must be typed into a page **without you ever
seeing the plaintext**. There is deliberately no method that returns a
secret's value — never try to read, print, or work around that.

- `generatePassword(name, length?)` — creates a random password and stores it
  as project secret `name`. Returns nothing.
- `setSecret(name, value)` — stores a value you already have. Returns nothing.
- `hasSecret(name)` — existence check (choose register vs. login flow, avoid
  overwriting an existing secret).
- `fillWithSecret(selector, name)` — types the secret into the matching field.
  This is the **only** way to use a secret. Use it instead of `page.type()`
  for any password field.

Register (generate + fill):

```bash
ws browser:eval "
    await page.goto('https://example.com/signup');
    await helper.generatePassword('EXAMPLE_COM_LOGIN');
    await page.type('#email', 'agent@example.com');
    await helper.fillWithSecret('#password', 'EXAMPLE_COM_LOGIN');
    await page.click('#submit');
"
```

Later login (reuse the same secret):

```bash
ws browser:eval "
    if (!await helper.hasSecret('EXAMPLE_COM_LOGIN')) return 'no secret';
    await page.goto('https://example.com/login');
    await page.type('#email', 'agent@example.com');
    await helper.fillWithSecret('#password', 'EXAMPLE_COM_LOGIN');
    await page.click('#submit');
"
```

Rules:
- Name secrets in UPPER_SNAKE_CASE, like env vars, per site/account (`EXAMPLE_COM_LOGIN`) and tell the user the
  name you created so they can find it.
- Never hardcode a password in script code or `page.type()` it, and never ask
  the user to paste one into chat. Use `generatePassword`, or ask the user to
  run `ws secret:create <name>` (it prompts for the value).
- Values resolved by `fillWithSecret` are scrubbed (exact match) from that
  invocation's stdout/stderr/return value, but **screenshots are not
  redacted** — don't screenshot a page showing a revealed password.
- Manage secrets from the CLI: `ws secret:create <name>`,
  `ws secret:inspect <name>`, `ws secret:rm <name>`.

## Before trying to launch the app yourself

The user typically already has the project's watch/dev command running in a
terminal (e.g. `composer run dev`, `ws exec npm run dev`). Check whether the
app is already reachable before starting a server — don't launch a second
dev server on top of one that's likely already running.

## If the browser isn't working

1. Try `ws browser:start` to (re)start the browser service.
2. If it still doesn't work after that, stop and tell the user — don't
   improvise a workaround (spinning up a separate headless browser, curling
   HTML and eyeballing it, etc.).