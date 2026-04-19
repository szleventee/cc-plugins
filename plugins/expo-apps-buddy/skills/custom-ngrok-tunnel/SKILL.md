---
name: custom-ngrok-tunnel
description: Run Expo Metro bundler behind a custom ngrok tunnel with a reserved domain. Use this skill when the user asks to "run expo through ngrok", "start expo with my custom tunnel", "use my ngrok domain with expo", "expo with a stable tunnel URL", or when Expo's built-in `--tunnel` flag fails. This is the reliable alternative to `npx expo start --tunnel` which bundles an ancient ngrok version that often breaks.
argument-hint: "[optional-ngrok-domain]"
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
  - Glob
  - Grep
---

# Custom ngrok Tunnel for Expo

Run Metro behind a user-managed ngrok v3 tunnel with a reserved domain. This is the **reliable** way to expose Expo over the internet — Expo's built-in `--tunnel` bundles `@expo/ngrok@2.3.41` (ancient, often errors with "Cannot read properties of undefined", random subdomains only).

A brew-installed ngrok v3 with a reserved domain:
- Is stable and doesn't crash on startup
- Keeps the same URL across restarts
- Works from anywhere (mobile data, different networks, corporate Wi-Fi)

## Prerequisites

### 1. ngrok v3+ installed

```bash
ngrok version
```

If missing or too old:
```bash
brew install ngrok
```

### 2. Authtoken configured

```bash
ls "$HOME/Library/Application Support/ngrok/ngrok.yml" 2>/dev/null
```

If missing, the user needs to sign up at ngrok.com (free tier works), grab their authtoken, then:
```bash
ngrok config add-authtoken <token>
```

### 3. Reserved domain

Free ngrok accounts get one reserved domain (e.g. `myapp.ngrok.app`). The user must create this at https://dashboard.ngrok.com/domains.

**Ask the user for their domain if they haven't provided one in `$ARGUMENTS`.** Store it in a variable for the rest of the session.

## Starting the Tunnel

### Step 1: Start ngrok on port 8081

```bash
# Use --url= (NOT the deprecated --domain=)
ngrok http --url=<domain> 8081 --log=stdout > /tmp/ngrok.log 2>&1 &
NGROK_PID=$!

# Give it a second to establish
sleep 3

# Verify ngrok is up
curl -s http://localhost:4040/api/tunnels | python3 -c "import sys,json; t=json.load(sys.stdin)['tunnels']; print(t[0]['public_url'] if t else 'no tunnel')"
```

If this fails, check `cat /tmp/ngrok.log` for the actual error.

### Step 2: Start Metro with tunnel env vars

Metro needs to advertise the tunnel hostname in its manifest, otherwise the phone will try to connect to `localhost` (which doesn't exist on the phone). Set two env vars:

- `REACT_NATIVE_PACKAGER_HOSTNAME` — the bare domain
- `EXPO_PACKAGER_PROXY_URL` — the full https URL

**Option A: Inline (one-shot)**

```bash
export EXPO_LOG_FILE="/tmp/expo-metro-$(date +%s).log"
REACT_NATIVE_PACKAGER_HOSTNAME=<domain> \
EXPO_PACKAGER_PROXY_URL=https://<domain> \
npx expo start --port 8081 > "$EXPO_LOG_FILE" 2>&1 &
EXPO_PID=$!

# Wait for Metro
for i in $(seq 1 20); do
  if grep -q "Metro waiting" "$EXPO_LOG_FILE" 2>/dev/null; then
    echo "Metro ready"; break
  fi
  sleep 1
done
```

**Option B: Persistent via `.claude/launch.json`**

If the user wants Claude Code's preview server to also use the tunnel (so the web preview MCP tools work against ngrok), update `.claude/launch.json`:

```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "expo-dev",
      "runtimeExecutable": "npx",
      "runtimeArgs": ["expo", "start", "--port", "8081"],
      "port": 8081,
      "autoPort": false,
      "env": {
        "REACT_NATIVE_PACKAGER_HOSTNAME": "<domain>",
        "EXPO_PACKAGER_PROXY_URL": "https://<domain>"
      }
    }
  ]
}
```

Note `autoPort: false` — ngrok is forwarding 8081, so Metro must stay on 8081.

### Step 3: Verify the tunnel works

```bash
# Metro packager status — should return "packager-status:running"
curl -s https://<domain>/status

# Manifest should show the tunnel hostname, not localhost
curl -s "https://<domain>/" -H "Expo-Platform: ios" | python3 -c "import sys,json; m=json.load(sys.stdin); print('manifest url:', m.get('launchAsset',{}).get('url',''))"
```

If `/status` returns anything other than `packager-status:running`, or the manifest URL still says `localhost`, the env vars didn't take effect. Restart Metro with the env vars set correctly.

### Step 4: Give the user the URL

Tell the user to open Expo Go and enter:

```
exp://<domain>
```

(No port needed — ngrok handles the 443 → 8081 mapping.)

## Monitoring

Both ngrok and Metro log to files — read them proactively:

```bash
# Metro output (JS errors, bundle progress, device connections)
tail -40 "$EXPO_LOG_FILE"

# ngrok traffic (HTTP requests, status)
tail -40 /tmp/ngrok.log

# Or check ngrok's web UI
curl -s http://localhost:4040/api/requests/http | python3 -m json.tool | head -60
```

## Stopping

```bash
kill $EXPO_PID 2>/dev/null
kill $NGROK_PID 2>/dev/null
kill $(lsof -ti :8081) 2>/dev/null
kill $(lsof -ti :4040) 2>/dev/null  # ngrok web UI
```

## Troubleshooting

- **"Cannot read properties of undefined"** when using `npx expo start --tunnel` — that's the bundled old ngrok. Use this skill instead.
- **Phone connects but app doesn't load** — manifest probably still points to localhost. Check `curl -s https://<domain>/ -H "Expo-Platform: ios"` and verify the env vars were set when Metro started.
- **ngrok errors on startup** — check `cat /tmp/ngrok.log`. Common causes: domain already in use by another ngrok session (kill it with `pkill ngrok`), wrong authtoken, domain not reserved on the account.
- **`--domain=` flag error** — use `--url=` instead, `--domain=` is deprecated in ngrok v3.
- **Port 8081 in use** — `kill $(lsof -ti :8081)` then retry.

## Why not use `npx expo start --tunnel`?

The built-in tunnel uses a bundled `@expo/ngrok@2.3.41`:
- Released in 2021, doesn't support modern ngrok API
- Random subdomains only (URL changes every restart)
- Frequently errors with "Cannot read properties of undefined" on modern ngrok accounts
- No control over the tunnel — can't see requests, can't pause, can't inspect

A user-managed ngrok v3 install sidesteps all of this.
