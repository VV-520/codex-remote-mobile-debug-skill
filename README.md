# Codex Remote Mobile Debug Skill

A Codex skill for diagnosing ChatGPT mobile remote-control connectivity to Codex Desktop on Windows.

The core lesson is simple: a live `codex remote-control` process only proves local process health. It does **not** prove that the phone, ChatGPT cloud path, plugin sync path, proxy route, or Cloudflare path is healthy.

## What this skill helps diagnose

Use this skill when:

- ChatGPT mobile says Codex is offline.
- `codex remote-control` appears to run, but the phone does not connect.
- Codex Cloud / Remote Control setup is confusing.
- A launcher keeps `remote-control` alive, but cloud sync still fails.
- Proxy or Cloudflare challenge responses break Codex remote connectivity.
- Local diagnostics say everything is running, but the actual mobile connection still fails.

## Included

- `SKILL.md`: the installable Codex skill.
- `.gitignore`: excludes logs, databases, environment files, and private/secrets files.
- `LICENSE`: MIT license.

## Installation

Use the installation flow supported by your Codex environment.

A typical local setup is:

1. Clone or download this repository.
2. Copy `SKILL.md` into the local Codex skills location used by your Codex build.
3. Restart or reload Codex so it can discover the skill.
4. Ask Codex to diagnose a mobile remote-control issue and reference this skill by name: `codex-remote-mobile-debug`.

If your Codex build uses a different skill import system, keep the file name and frontmatter intact and import `SKILL.md` through that system.

## Example prompt

```text
My ChatGPT mobile app says Codex is offline, but on Windows I can see codex remote-control running. Diagnose the issue with the codex-remote-mobile-debug skill.
```

The skill should not stop at “the process is alive.” It should separate:

1. Local Codex Desktop state.
2. Local `remote-control` process or launcher state.
3. Mobile/account/cloud evidence.
4. Network, proxy, and ChatGPT endpoint evidence.
5. Next action and risk.

## Expected output pattern

A good diagnosis should say something like:

```text
Local Codex Desktop: signed in / not signed in / unknown.
Remote-control process: running / not running / exited / launcher-only evidence.
Mobile/cloud evidence: fresh mobile records found / no fresh records / account state unclear.
Network/proxy evidence: direct route works / proxy route fails / Cloudflare challenge detected.
Next action: smallest safe fix, plus what not to change yet.
```

The important rule is: do not call the connection healthy unless both local process health and mobile/cloud-path evidence are present.

## Privacy and safety

This skill is written as a diagnostic workflow. It does not include private logs, account emails, access tokens, cookies, or user-specific absolute paths.

Before publishing forks, examples, logs, screenshots, or troubleshooting reports, remove:

- account emails
- usernames and device names
- absolute home-directory paths
- request cookies
- API keys and bearer tokens
- raw `logs_2.sqlite` rows
- private ChatGPT conversation content

Keep diagnostic commands read-only by default. Ask before restarting proxy clients, killing processes, editing network settings, or changing scheduled tasks.

## License

MIT
