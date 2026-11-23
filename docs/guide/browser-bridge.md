# Browser Bridge Setup

The Browser Bridge is the default connection for commands that need a browser. If a command requires login, sign in to the target site in the Chrome profile where the extension is enabled. Public API commands such as `hackernews top` do not need the bridge.

OpenCLI connects to your browser through a lightweight **Browser Bridge** Chrome Extension + micro-daemon (zero config, auto-start).

## Extension Installation

### Method 1: Download Pre-built Release (Recommended)

1. Go to the GitHub [Releases page](https://github.com/jackwener/opencli/releases) and download the latest `opencli-extension.zip`.
2. Unzip the file and open `chrome://extensions`, enable **Developer mode** (top-right toggle).
3. Click **Load unpacked** and select the unzipped folder.

### Method 2: Load Unpacked Source (For Developers)

1. Open `chrome://extensions` and enable **Developer mode**.
2. Click **Load unpacked** and select the `extension/` directory from the repository.

## Verification

The daemon starts on demand when a command uses the Browser Bridge. `opencli doctor` can also start it if needed, but it does not install or enable the extension.

```bash
opencli doctor             # Check the daemon, extension, and live browser connection
opencli doctor --no-live   # Skip the live browser test; may still start the daemon
opencli doctor --sessions  # Also list automation workspaces if the extension is connected
```

## How It Works

```
opencli CLI <--- HTTP ---> daemon <--- WebSocket ---> Browser Bridge extension
                       127.0.0.1:19825                 (in your Chrome profile)
```

The CLI sends requests to the loopback daemon over HTTP. The daemon forwards them over the extension's WebSocket connection and returns the results to the CLI.

## Daemon Lifecycle

The daemon exits automatically once it has received no browser commands for **4 hours** and the Chrome extension is disconnected. It can stay running longer while the extension is connected. Checking status or logs does not reset the idle timer.

```bash
opencli daemon status    # Check daemon state (PID, uptime, extension, memory)
opencli daemon stop      # Graceful shutdown
opencli daemon restart   # Stop + restart
```

Override the timeout via the `OPENCLI_DAEMON_TIMEOUT` environment variable (milliseconds). Set to `0` to keep the daemon alive indefinitely.
