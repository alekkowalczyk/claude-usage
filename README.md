# claude-usage

One terminal view of your Claude subscription usage limits across **multiple accounts** — work orgs and personal plans side by side. Comes with a local web dashboard and a live list of your running Claude Code sessions.

```
company-org1  (you@company.com · Org One)
  Session (5h)        ██░░░░░░░░░░░░░░░░░░   8%   resets in 2h 18m (Wed 09:08 PM)
  Week (all models)   ███████░░░░░░░░░░░░░  34%   resets in 4d 4h (Sun 10:50 PM)
  Week (Opus)         ████████████████░░░░  81%   resets in 4d 4h (Sun 10:50 PM)
  Extra usage (mo)    ██████████░░░░░░░░░░  50%   175.86 USD of 350.00 USD

company-org2  (you@company.com · Org Two)
  Session (5h)        ░░░░░░░░░░░░░░░░░░░░   0%
  Week (all models)   █░░░░░░░░░░░░░░░░░░░   3%   resets in 2d 4h (Fri 10:50 PM)

personal-1  [cached 45s ago]
  Session (5h)        ████████████░░░░░░░░  62%   resets in 30m (Wed 07:20 PM)
  Week (all models)   ██████████████████░░  91%   resets in 12h 0m (Thu 06:50 AM)
```

If you juggle several Claude subscriptions (a couple of company orgs, a personal Max plan…), checking how much headroom each one has left means logging in and out or hopping between machines. This tool stores an OAuth token per account and shows all the usage windows — the 5-hour session, the weekly all-models / Opus / Sonnet limits, and extra-usage spend — in one place.

> **Unofficial.** This uses the same OAuth flow and usage endpoint as the Claude Code CLI (`api.anthropic.com/api/oauth/usage`). The endpoints are undocumented, not supported by Anthropic, and may change or break at any time. Not affiliated with Anthropic.

## Features

- **Multiple accounts** — label them however you like (`work-org1`, `personal`, …), each with its own OAuth token that auto-refreshes.
- **All usage windows** — 5-hour session, weekly (all models / Opus / Sonnet / apps), and monthly extra-usage spend, with reset times.
- **Web dashboard** — `claude-usage serve` gives you a self-refreshing localhost page with the same data.
- **Running sessions** (macOS) — `claude-usage sessions` lists every running Claude Code process, which account and config dir it uses, its project directory, and when it was last active — sorted most-recent first, with an **● active** badge for sessions that did something in the last 15 minutes.
- **Polite to the API** — results are cached for 2 minutes and fetches are spaced out, because the usage endpoint rate-limits aggressively.
- **Zero dependencies** — a single executable Node script.

## Install

Requires [Node.js](https://nodejs.org) 18 or newer. No `npm install` needed.

```sh
git clone https://github.com/alekkowalczyk/claude-usage.git
ln -s "$PWD/claude-usage/claude-usage" /usr/local/bin/claude-usage
```

(or copy the `claude-usage` file anywhere on your `PATH` and `chmod +x` it).

## Usage

```
claude-usage              show all accounts (first run: guided login)
claude-usage login NAME   add / re-authenticate an account
claude-usage remove NAME  forget an account
claude-usage list         list stored accounts
claude-usage sessions     running Claude Code sessions, most recently active first
claude-usage serve [port] web dashboard on http://localhost:7788 (localhost only)
claude-usage --json       machine-readable output
claude-usage --fresh      bypass the 2-minute result cache
claude-usage --demo       render sample output (no network, no login)
```

On first run it walks you through logging in each account: it opens the standard `claude.ai` OAuth consent page in your browser and asks you to paste back the authorization code. To add an account under a different claude.ai login (or to pick a specific org), open the printed URL in a private window or another browser profile.

### Web dashboard

```sh
claude-usage serve          # http://localhost:7788
```

Usage meters for every account plus the running-sessions table, auto-refreshing (usage every ~3 minutes to respect rate limits, sessions every 15 seconds). Light and dark mode. The server binds to `127.0.0.1` only — it exposes account emails and org names, so it is deliberately not reachable from the network.

### Sessions

`claude-usage sessions` (and the dashboard's sessions table) finds running `claude` processes, resolves each one's `CLAUDE_CONFIG_DIR` to figure out which account it is logged into, and reads the project's transcript files to determine when the session last did anything. Sessions active within the last 15 minutes get a green **●** badge. macOS only for now (it shells out to `ps` and `lsof`).

## How it works, and where your tokens live

- Login uses the Claude Code CLI's public OAuth client with PKCE — this tool never sees your password, only the resulting tokens.
- Tokens are stored in `~/.config/claude-usage/profiles.json`, created with mode `600` (owner read/write only). `claude-usage remove NAME` deletes an account's tokens; revoke sessions at [claude.ai/settings](https://claude.ai/settings) for extra certainty.
- Usage responses are cached in `~/.config/claude-usage/cache.json` for 2 minutes; if a refresh fails you'll see the last known data marked with a warning instead of nothing.
- Nothing ever leaves your machine except the OAuth and usage requests to Anthropic's own endpoints.

## Caveats

- Undocumented endpoints — any part of this can break without notice when Anthropic changes them.
- The usage endpoint rate-limits hard (~3 minutes is the community-reported safe polling interval). If you get rate-limited, wait a few minutes; cached data keeps rendering meanwhile.
- Session detection is macOS-only and infers "last active" from transcript file mtimes — sessions sharing the same project directory share that signal.

## License

[MIT](LICENSE)
