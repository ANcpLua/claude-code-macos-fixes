# Claude Code macOS Fixes

Solutions for two problems Claude Code currently has on macOS, debugged live on a real machine (MacBook Air M4, macOS Tahoe 26.5.2, Ghostty 1.3.1, Homebrew install of `@anthropic-ai/claude-code`).

📄 **Full guide:** [ancplua.github.io/claude-code-macos-fixes](https://ancplua.github.io/claude-code-macos-fixes/) · source: [`computer-use-tcc-fix.html`](computer-use-tcc-fix.html)

## The two problems

Computer Use in the Claude Code CLI loops forever with *"Accessibility and Screen Recording permission(s) not yet granted"* — no matter which app toggles you flip in System Settings. Two macOS facts cause it:

1. **TCC binds permissions to the requesting binary, not the host app.** The built-in `computer-use` MCP server runs *inside* the `claude` CLI process, so macOS attributes Accessibility and Screen Recording to `claude.exe` (yes, really — it's a native macOS binary with a leftover `.exe` suffix), not to Ghostty/iTerm/Terminal and not to Claude.app. Terminal-app grants cover other things; your shell's `screencapture` may even work while the MCP server stays blocked.

2. **TCC grants are only read at process start.** A running `claude` process caches its permission state — fixing the lists mid-session changes nothing until you restart the CLI. This is why "I granted it and it still fails" is the default experience.

## The fix

```sh
# 1 — where is the binary really? (works for brew, npm, and bun installs)
readlink -f $(which claude)
# → /opt/homebrew/lib/node_modules/@anthropic-ai/claude-code/bin/claude.exe

# 2 — reveal it in Finder (Spotlight does NOT index /opt/homebrew)
open -R "$(readlink -f $(which claude))"
```

3. System Settings → Privacy & Security → drag `claude.exe` from Finder into **Accessibility** AND **Screen & System Audio Recording**, enable both toggles. Drag & drop skips the broken "+" dialog entirely.

4. Restart the CLI — context survives:

```sh
# Ctrl+C, then:
claude --resume   # pick your session — nothing is lost
```

5. Have Claude call `request_access` again → the one-time per-app approval dialog appears → approve → computer use is live.

The fix also holds **inside tmux**, because it targets the `claude` binary itself.

## The traps

- **The "+" dialog search field is Spotlight** — it will never find `/opt/homebrew`. Drag & drop from Finder instead.
- **Don't double-click `claude.exe`** — macOS treats `.exe` as a Windows archive and hands it to The Unarchiver. Cancel; the file is dragged, never opened.
- **Granting the wrong app** — Ghostty/iTerm/Claude.app entries cover other features; quick shell sanity checks can "prove" permissions are fine when they aren't.
- **The stale process** — everything correct, still failing? The session predates the grant. Restart with `claude --resume`.
- **The orphaned per-version helper** (community-reported, unverified here) — see [anthropics/claude-code#50735](https://github.com/anthropics/claude-code/issues/50735); a last resort only, after ruling out the traps above.

See the [HTML guide](computer-use-tcc-fix.html) for the full walkthrough, before/after transcripts, and FAQ.
