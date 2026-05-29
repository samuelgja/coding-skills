<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.svg">
    <img alt="coding-skills" src="assets/logo.svg" width="380">
  </picture>
</p>

<p align="center"><strong>Write code that passes review the first time.</strong></p>

<p align="center">
Three agent skills that learn your team's real conventions from its pull-request comments —<br>
so a coworker or lead never has to leave the same comment twice.
</p>

<p align="center">
  <a href="https://skills.sh/samuelgja/coding-skills"><img alt="skills.sh" src="https://skills.sh/b/samuelgja/coding-skills"></a>
  <img alt="MIT" src="https://img.shields.io/badge/license-MIT-1B9E77">
</p>

## Install

```bash
npx skills add samuelgja/coding-skills
```

Works with Claude Code, Cursor, Codex, Copilot, Windsurf, Gemini, and more — the CLI auto-detects your agent.

## The three skills

| Skill | What it does |
|-------|--------------|
| **learn-coding** | Reads your repo's code and **PR review comments** (weighting owners & maintainers) and writes the team's real rules to `.coding/`. Run once; re-run to refresh. |
| **coding** | Applies those rules — on top of a strict baseline — every time you write code, so it passes review the first time. |
| **fix-me-bug** | Test-first bug fixing that follows the same rules, so the fix doesn't draw new comments. |

## How it works

```mermaid
flowchart LR
    SRC["Your repo<br/><small>code · config · PR review comments</small>"]
    LC(["learn-coding"])
    DOC[".coding/<br/>coding-guidelines.md"]
    CODE(["coding"])
    FIX(["fix-me-bug"])
    OUT["Review-ready code<br/><small>no repeat comments</small>"]

    SRC -->|"mine · weight maintainers"| LC
    LC -->|writes| DOC
    DOC -->|new code| CODE
    DOC -->|bug fixes| FIX
    CODE --> OUT
    FIX --> OUT

    classDef skill fill:#1B9E77,stroke:#15805d,color:#F6F8F6,stroke-width:1px;
    classDef doc fill:#E8F5EF,stroke:#1B9E77,color:#16181D,stroke-width:1px;
    classDef io fill:#F4F5F4,stroke:#CBD2CE,color:#16181D,stroke-width:1px;
    class LC,CODE,FIX skill;
    class DOC doc;
    class SRC,OUT io;
```

Commit `.coding/` so the whole team shares one standard. No learned file yet? `coding` and `fix-me-bug` still work on a strict built-in baseline — run `learn-coding` to make them yours.

> `learn-coding` uses the GitHub CLI (`gh`). The other two need nothing.

## License

MIT
