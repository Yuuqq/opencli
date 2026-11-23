# Remote Chrome

Run OpenCLI on a server or headless environment by connecting to a remote Chrome instance.

## Use Cases

- Running CLI commands on a remote server
- CI/CD automation with headed browser
- Shared team browser sessions

## Setup

### 1. Start Chrome on the Remote Machine

```bash
# On the remote machine (or your Mac)
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/chrome-debug-profile"
```

### 2. SSH Tunnel (If Needed)

If the remote Chrome is on a different machine, create an SSH tunnel:

```bash
# On your local machine or server
ssh -L 9222:127.0.0.1:9222 user@remote-host
```

::: warning
Use `127.0.0.1` instead of `localhost` in the SSH command to avoid IPv6 resolution issues that can cause timeouts.
:::

### 3. Configure OpenCLI

```bash
export OPENCLI_CDP_ENDPOINT="http://127.0.0.1:9222"
```

### 4. Verify

```bash
# Test the connection
curl http://127.0.0.1:9222/json/version

# Run a browser command against the configured CDP endpoint
opencli bilibili hot --limit 5
```

`opencli doctor` checks the local Browser Bridge, not the direct CDP endpoint. Sign in to the target site in the debug profile if the command requires login.

## CI/CD Integration

For CI/CD environments, use a real Chrome instance with `xvfb`:

::: v-pre
```yaml
steps:
  - uses: browser-actions/setup-chrome@latest
    id: setup-chrome
  - run: |
      xvfb-run --auto-servernum \
        ${{ steps.setup-chrome.outputs.chrome-path }} \
        --remote-debugging-port=9222 \
        --user-data-dir="$(mktemp -d)" &
```
:::

Point OpenCLI at the running browser:
::: v-pre
```yaml
env:
  OPENCLI_CDP_ENDPOINT: http://127.0.0.1:9222
```
:::
