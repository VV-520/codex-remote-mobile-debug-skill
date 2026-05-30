---
name: codex-remote-mobile-debug
description: Use when diagnosing or repairing Codex mobile or ChatGPT remote-control connectivity on Windows, including ChatGPT mobile cannot connect to Codex, /remote-control, codex remote-control appears alive but the phone is offline, Codex Cloud/Remote setup confusion, launcher/shortcut lifecycle issues, and proxy or Cloudflare network failures such as HTTP_PROXY, local proxy listeners, 403 Forbidden, or Cf-Mitigated challenge.
metadata:
  short-description: Diagnose Codex mobile remote control on Windows
---

# Codex Remote Mobile Debug

Use this skill when the user is trying to connect ChatGPT mobile or ChatGPT remote control to Codex Desktop, especially on Windows.

The key lesson: `codex remote-control` being alive only proves local process health. It does not prove the phone, ChatGPT cloud path, or plugin sync path is healthy.

## Mental model

Diagnose in four layers. Do not skip directly to script edits.

1. **Mobile/account layer**
   - ChatGPT mobile must be logged into the account that is allowed to control Codex.
   - Do not promise to bypass a required mobile login or authorization step.
   - If the phone says Codex is offline, treat that as a cloud-path symptom, not just a local process symptom.

2. **Codex Desktop setup layer**
   - Confirm Codex Desktop is installed and signed in.
   - Confirm Codex Cloud / Remote Control setup is complete.
   - Do not confuse the app's SSH connections screen with ChatGPT mobile remote control. SSH connections are for connecting from this PC to remote machines; they are not the phone-to-Codex remote-control setup.

3. **Local process and launcher layer**
   - Check Codex Desktop, `codex.exe remote-control`, launcher, scheduled tasks, shortcuts, logs, and the user's custom launcher if present.
   - A local summary file is useful, but it only proves local launcher/process state.

4. **Cloud/proxy/network layer**
   - If local processes are healthy but phone still shows offline, inspect the actual network path to `chatgpt.com` and Codex backend endpoints.
   - Proxy or Cloudflare challenges can make the phone appear offline even while `remote-control` is running.

## Fast triage

Start with the user's actual symptom:

- If the user says "手机端离线", "ChatGPT 远程连接 Codex 没反应", or "不对，肯定有异常", do not stop after seeing a live process.
- If the user asks where to configure it, open or guide them to Codex connection settings, not SSH connections.
- If the user asks why it died, distinguish "launcher not running", "remote-control exited", "Codex app exited", and "cloud path blocked".

Run read-only checks first:

```powershell
codex login status
codex cloud list
codex remote-control --help
Get-Process Codex,codex -ErrorAction SilentlyContinue | Select-Object Id,ProcessName,Path
Get-ScheduledTask -TaskName 'Codex Remote Control' -ErrorAction SilentlyContinue | Select-Object TaskName,State,Description
```

If a machine has a custom launcher under `$HOME\.codex\remote-control`, prefer its status files:

```powershell
$rc = Join-Path $env:USERPROFILE '.codex\remote-control'
if (Test-Path (Join-Path $rc 'check-codex-remote-control.ps1')) {
  & (Join-Path $rc 'check-codex-remote-control.ps1')
}
if (Test-Path (Join-Path $rc 'remote-control-summary.txt')) {
  Get-Content (Join-Path $rc 'remote-control-summary.txt')
}
```

Interpretation:

- `codex login status` showing ChatGPT login is necessary but not sufficient.
- `codex cloud list` returning no tasks does not prove mobile remote control is healthy.
- `remote-control.err.log` and `remote-control.out.log` being empty does not prove cloud connectivity.
- `remote-control-summary.txt` can say "正常运行" while the phone is still offline if the cloud/proxy path is blocked.

## Network and proxy checks

Run these when local state looks healthy but mobile is offline:

```powershell
Get-ChildItem Env:HTTP_PROXY,Env:HTTPS_PROXY,Env:ALL_PROXY,Env:NO_PROXY -ErrorAction SilentlyContinue |
  Select-Object Name,Value

netsh winhttp show proxy

Get-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' |
  Select-Object ProxyEnable,ProxyServer,AutoConfigURL

Resolve-DnsName chatgpt.com -ErrorAction SilentlyContinue |
  Select-Object Name,Type,IPAddress,NameHost

Test-NetConnection chatgpt.com -Port 443 |
  Select-Object ComputerName,RemoteAddress,TcpTestSucceeded
```

If an environment proxy points to a local port, identify the owner:

```powershell
netstat -ano | Select-String -Pattern ':10793'
Get-Process -Id <PID> -ErrorAction SilentlyContinue | Select-Object Id,ProcessName,Path
```

Check ChatGPT endpoint behavior:

```powershell
curl.exe -I https://chatgpt.com/
curl.exe -I 'https://chatgpt.com/backend-api/plugins/featured?platform=codex'
curl.exe --noproxy "*" -I https://chatgpt.com/
```

Important signals:

- DNS success and TCP 443 success only prove basic reachability.
- `403 Forbidden` with `Cf-Mitigated: challenge` and `Server: cloudflare` is a strong sign the ChatGPT/Codex cloud path is blocked or challenged.
- A real failure pattern is `HTTP_PROXY`, `HTTPS_PROXY`, and `ALL_PROXY` pointing to a local proxy such as `http://127.0.0.1:10793`, while the proxy route returns Cloudflare challenge responses for ChatGPT endpoints.
- A proxy challenge on `chatgpt.com` can break mobile/remote/plugin sync even if local Codex processes are alive.

## Codex logs

When process and network checks disagree, inspect Codex logs. A common local path is:

```text
%USERPROFILE%\.codex\logs_2.sqlite
```

Look for terms like:

- `codex_chatgpt_android_remote`
- `remote plugin sync request`
- `backend-api/plugins`
- `403 Forbidden`
- `Cloudflare`
- `Cf-Mitigated`

Fresh `codex_chatgpt_android_remote` entries are a stronger signal than a live local `remote-control` process. Old entries with no current mobile connection records support the conclusion that the phone/cloud path is not currently connected.

## Common fixes and boundaries

Use the smallest safe fix for the failing layer:

- **Setup missing**: open Codex connection settings with `Start-Process codex://settings/connections` or guide the user to the correct desktop setting. Do not send them to SSH connections for phone remote control.
- **Local remote-control not running**: start `codex remote-control` or use the user's launcher shortcut. If using scripts with Chinese strings under Windows PowerShell 5.1, save them as UTF-8 with BOM.
- **Launcher lifecycle issue**: if the user wants remote-control only while Codex is open, prefer a desktop shortcut or launcher that starts with Codex and exits when Codex exits. Avoid a login-triggered scheduled task unless the user explicitly wants it.
- **Quiet status needed**: write human-readable Chinese summary to a separate `remote-control-summary.txt`; keep technical logs separate.
- **Cloud/proxy blocked**: change proxy node, bypass the failing proxy, test cellular hotspot, or fix the local proxy route. Ask before killing or restarting proxy processes because that can affect the whole PC.
- **Mobile requires login**: do not claim Codex can skip ChatGPT mobile authorization. The user must complete required account auth.

## Open-source hygiene

Before publishing this skill or output derived from it:

- Remove personal usernames, account emails, device names, absolute home-directory paths, screenshots, private logs, tokens, and request cookies.
- Express home paths with `$env:USERPROFILE`, `%USERPROFILE%`, or `$HOME`.
- Treat local proxy process names and ports as examples, not universal facts.
- Do not include raw `logs_2.sqlite` rows unless they have been reviewed and redacted.
- Do not include API keys, bearer tokens, cookies, `Authorization` headers, or private ChatGPT conversation content.
- Keep diagnostic commands read-only by default. Ask before restarting proxy clients, killing processes, editing network settings, or changing scheduled tasks.

## Reporting pattern

Report with this separation, using the user's language:

1. Local Codex Desktop state.
2. Local `remote-control` process/launcher state.
3. Mobile/cloud evidence.
4. Network/proxy evidence.
5. Next action and risk.

Use concrete evidence:

- PIDs and process names.
- Scheduled task state.
- Summary file state.
- Proxy env vars.
- `curl` status codes and key headers.
- Recent log records or absence of fresh mobile records.

Avoid saying "remote-control is healthy" unless both local process health and the cloud/mobile path have evidence.
