# Wocker plugin for Claude Code

Skills that help Claude Code work with [Wocker](https://github.com/kearisp/wocker-area) projects.

## Install

```shell
claude plugin marketplace add kearisp/claude-wocker-plugin
claude plugin install wocker@wocker
```

Or from inside a session:

```
/plugin marketplace add kearisp/claude-wocker-plugin
/plugin install wocker@wocker
```

## Skills

- `/wocker:browser` — drive the browser built into the `ws` CLI (`ws browser:eval`, `ws browser:exec`,
  `ws browser:screenshot`), including the `helper` API for project env, meta and secrets.
  Claude also picks it up automatically when you ask it to open, click or screenshot a page.

## Updating

Bump `version` in `.claude-plugin/plugin.json` on every release, otherwise users won't receive the update:

```shell
claude plugin update wocker@wocker
```
