# Connecting OpenCLI via CDP (Remote/Headless Servers)

If you cannot use the opencli Browser Bridge extension (e.g., in a remote headless server environment without a UI), OpenCLI provides an alternative: connecting directly to Chrome via **CDP (Chrome DevTools Protocol)**.

Because CDP binds to `localhost` by default for security reasons, accessing it from a remote server requires an additional networking tunnel.

This guide is broken down into three phases:
1. **Preparation**: Start Chrome with CDP enabled locally.
2. **Network Tunnel**: Reach the loopback port from your remote server through SSH.
3. **Execution**: Run OpenCLI on your server.

---

## Phase 1: Preparation (Local Machine)

First, you need to start a Chrome browser on your local machine with remote debugging enabled.

**macOS:**
```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/chrome-debug-profile"
```

**Linux:**
```bash
google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/chrome-debug-profile"
```

**Windows:**
```cmd
"C:\Program Files\Google\Chrome\Application\chrome.exe" ^
  --remote-debugging-port=9222 ^
  --user-data-dir="%USERPROFILE%\chrome-debug-profile"
```

Once this browser instance opens, **log into the target websites you want to use** (e.g., bilibili.com, zhihu.com) so that the session contains the correct cookies.

---

## Phase 2: SSH Tunnel

Keep the debugging port private: CDP can control the browser session. Do not expose it through a public listener or unauthenticated proxy.

Run this command on your **Local Machine** to forward the remote server's port `9222` back to your local port `9222`:

```bash
ssh -N -R 127.0.0.1:9222:127.0.0.1:9222 your-server-user@your-server-ip
```

Leave this SSH session running in the background.

## Phase 3: Execution (Remote Server)

On the remote server, check the forwarded endpoint, then try a command that uses the browser:

```bash
export OPENCLI_CDP_ENDPOINT="http://127.0.0.1:9222"
curl --fail "$OPENCLI_CDP_ENDPOINT/json/version"
opencli bilibili hot --limit 5
```

`opencli doctor` checks the local Browser Bridge, not the direct CDP endpoint.

> *Tip: If you provide a standard HTTP/HTTPS CDP endpoint, OpenCLI requests the `/json` target list and picks the most likely inspectable app/page target automatically. If multiple app targets exist, you can further narrow selection with `OPENCLI_CDP_TARGET` (for example `antigravity` or `codex`).*

If you plan to use this setup frequently, you can persist the environment variable by adding the `export` line to your `~/.bashrc` or `~/.zshrc` on the server.
