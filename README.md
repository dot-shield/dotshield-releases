# dotshield

**Keeps secrets away from AI coding agents.** DotShield stops Claude Code and Cursor from reading or printing your secrets (API keys, tokens, `.env` files, private keys), and includes a free offline scanner that finds the secrets already sitting in your projects, your git history and your agents' transcripts.

> **Public beta.** This is `0.1.0-beta`. It works, it is tested, and we want your feedback. See [The beta](#the-beta) below for what we ask of testers.

## What it does

- **Scan:** finds secrets in a folder, its git history and your AI agent files (Claude Code transcripts and settings, Cursor MCP config). Works offline and never prints a secret value.
- **Fix:** redacts secrets in agent transcripts (after a private backup), adds secret files to `.gitignore` and `.cursorignore`, adds Claude Code deny rules, and lists every secret you should rotate. It shows the plan and asks first.
- **Guard:** runs as Claude Code (or Cursor) hooks:
  - denies reads of secret files;
  - masks secrets in command output;
  - stops prompts that contain a secret;
  - masks secrets left in transcripts when a session ends.
- **Pre-commit:** stops git commits that contain secrets.

It starts in **watch mode**, which only logs what it would have done and blocks nothing, so you can see it work before you let it block.

## Install

Needs macOS, Linux or Windows, on x64 or arm64.

```bash
npx dotshield@beta scan             # try it without installing (Node 18 or later)
npm install -g dotshield@beta       # or install the dotshield command
brew install dot-shield/dotshield/dotshield   # or with Homebrew (macOS, Linux)
```

## Quick start

```bash
dotshield scan                      # find secrets here, in git history and in agent files
dotshield fix                       # clean them up (shows the plan, asks first)
dotshield guard install             # guard Claude Code (starts in watch mode)
dotshield guard install --agent cursor   # or Cursor; --agent both for both
dotshield guard status              # where it's installed, its mode, where it logs
dotshield guard report              # what it did, or would have done
dotshield guard mode enforce        # start blocking
```

New Claude Code and Cursor sessions pick up the guard. Sessions that are already open don't.

## Commands

| command | what it does |
| --- | --- |
| `dotshield scan [path]` | find secrets. `--json` or `--format sarif` for tools, `--no-history` to skip git history |
| `dotshield fix [path]` | redact agent transcripts, add ignore and deny rules, list secrets to rotate. `--dry-run` to only show the plan |
| `dotshield guard install` | add the guard hooks. `--agent cursor` or `--agent both` |
| `dotshield guard mode watch\|enforce` | log only, or block |
| `dotshield guard report` | sum up what the guard did. `--days 14 --json` for the beta report |
| `dotshield guard check` | show what the guard would do with a read or a command |
| `dotshield guard uninstall` | remove the hooks |
| `dotshield precommit install` | add a git pre-commit hook to the current repository |
| `dotshield allow fingerprint <fp>` | a finding isn't a secret: stop flagging it (the 12 characters DotShield prints) |
| `dotshield allow path <path>` | let agents read a file that looks secret but isn't, such as a test fixture |
| `dotshield allow list` | show the allowlist |
| `dotshield redact [file]` | print a file or stdin with its secrets masked |
| `dotshield version` | print the version |

Agents can't run `allow`, `guard mode` or `guard uninstall`: DotShield denies them, so an agent can't switch it off for you.

## The beta

Thank you for trying it. The beta takes two weeks:

1. **Week 1, watch.** Run `dotshield guard install` and work as usual. Now and then, run `dotshield guard report` to see what it would have blocked.
2. **Week 2, enforce.** Run `dotshield guard mode enforce`. If it blocks something that isn't a secret, use `dotshield allow fingerprint <fp>` or `dotshield allow path <path>`. That is also how the beta counts false blocks. If it gets in your way, `dotshield guard mode watch` turns blocking off. Please tell us why.
   After every scan, DotShield prints a short summary between two markers: counts per kind of secret, with no file names, paths or values. Paste it into the [beta report form](https://github.com/dot-shield/dotshield-releases/issues/new?template=beta-report.yml) and tell us which findings were wrong. (This needs DotShield 0.1.0-beta.5 or later.)
3. **At the end, send your report.**

   ```bash
   dotshield guard report --days 14 --json > dotshield-beta.json
   ```

   Read it first: it holds only counts, rule ids, dates and timings (how many calls were checked, DotShield's time per call, blocks, how many you allowlisted, when you changed mode). It never holds a file path, a file's contents or a secret value. **Don't send** the log behind it (`~/.dotshield/log/guard.jsonl`), because it contains file paths.

   **Send it with the [beta report form](https://github.com/dot-shield/dotshield-releases/issues/new?template=beta-report.yml)**, along with anything that surprised you, got in your way or was missing. Reports are public GitHub issues, which is why the report holds nothing sensitive. Bugs and questions are welcome any time, [as an issue](https://github.com/dot-shield/dotshield-releases/issues).

## Privacy

DotShield has no network code. Nothing you scan or guard leaves your machine, and the beta report goes out only when you send it yourself. The one network call is the first-run download of the binary, described below.

## How the binary gets to you

This npm package is a small wrapper with no dependencies and no install script. On first use it downloads the `dotshield` binary for your system from [the GitHub release](https://github.com/dot-shield/dotshield-releases/releases) in this repository of the same version. It checks the binary's SHA-256 against checksums taken from the release's signed `checksums.txt` and published inside this package, and keeps it in `~/.dotshield/bin/`. It checks the binary again on every run. The agent hooks call the binary directly, so Node isn't on the hook path.

## Uninstall

```bash
dotshield guard uninstall           # remove the hooks from Claude Code (--agent cursor or both)
npm uninstall -g dotshield          # or: brew uninstall dotshield
rm -rf ~/.dotshield                 # optional: the binary, your allowlist and the local log
```

## Licence

Free to use, including at work, under the [DotShield Free Use Licence](LICENSE): no redistribution, modified copies, reverse engineering or competing products. Versions 0.1.0-beta.1 and 0.1.0-beta.2 were released under Apache-2.0. Third-party notices: [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

This repository holds release files only; DotShield's source code is not public.

DotShield is not affiliated with or endorsed by Anthropic or Cursor.
