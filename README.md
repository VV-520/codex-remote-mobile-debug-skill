# Codex Remote Mobile Debug Skill

This repository contains a Codex skill for diagnosing ChatGPT mobile remote-control connectivity to Codex Desktop on Windows.

The core lesson is simple: a live `codex remote-control` process does not prove that the mobile or cloud path is healthy. The skill guides an agent to check local process health, Codex setup, mobile/account state, and the real `chatgpt.com` proxy or Cloudflare path separately.

## Included

- `SKILL.md`: the installable Codex skill.

## Use Cases

- ChatGPT mobile says Codex is offline.
- `/remote-control` appears to run, but the phone does not connect.
- Codex Cloud / Remote Control setup is confusing.
- A local launcher keeps `remote-control` alive, but cloud sync still fails.
- Proxy or Cloudflare challenge responses break Codex remote connectivity.

## Privacy

This skill is written as a diagnostic workflow. It does not include private logs, account emails, access tokens, cookies, or user-specific absolute paths.

Before publishing forks or examples, review any added logs or screenshots for personal information and credentials.

## License

No license has been selected yet.
