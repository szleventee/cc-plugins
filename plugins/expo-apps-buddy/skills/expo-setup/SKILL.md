---
name: expo-setup
description: Set up a complete Expo React Native project with a sample app in the current folder. Great for beginners. Uses expo:building-native-ui best practices.
argument-hint: "[optional-app-name]"
allowed-tools:
  - Bash
  - Write
  - Edit
  - Read
  - Glob
  - mcp__Claude_Preview__preview_start
  - mcp__Claude_Preview__preview_screenshot
  - mcp__Claude_Preview__preview_logs
  - mcp__Claude_Preview__preview_resize
---

# Expo Project Setup Skill

Set up a complete Expo React Native + TypeScript project in the current directory with a polished sample app, all dependencies, preview config, and beginner-friendly guidance.

**This skill uses the `expo:building-native-ui` skill as the foundation for all UI code.** Always load that skill when building or modifying app UI after setup.

## Arguments

- `$ARGUMENTS` — Optional project/app name. If empty, use the current folder name.

## Instructions

Follow these steps **exactly in order**. Do NOT skip any step.

### Step 1: Check Prerequisites

Run `node --version` and `npm --version`. If Node.js is not installed, stop and tell the user:
> "You need Node.js installed first. Run `brew install node` in your terminal, then try again."

### Step 2: Create the Expo Project

If the current folder is empty (no package.json), scaffold a new project.

**IMPORTANT:** The directory will often already contain a `.claude/` folder or `.git/` — `create-expo-app` will refuse to scaffold into a non-empty directory. Always use the temp directory approach:

```bash
npx create-expo-app@latest expo-temp-project --template blank-typescript
```

Then move all files into the current directory using `cp -a` (which handles dotfiles correctly in zsh, unlike `mv` with `shopt -s dotglob` which is bash-only):

```bash
cp -a expo-temp-project/. .
rm -rf expo-temp-project
```

After creation, run `npm install` to make sure node_modules is correct.

### Step 3: Install Libraries

Install core libraries plus extras for proper native-quality UI:

```bash
npx expo install expo-router expo-linear-gradient expo-status-bar expo-haptics expo-image react-native-safe-area-context react-dom react-native-web @expo/metro-runtime
npm install @expo/ngrok
```

**Library choices (from building-native-ui guidelines):**
- `expo-router` — native navigation stacks, tabs, and routing
- `expo-image` — use for SF Symbols on native (`source="sf:name"`) and optimized images
- `expo-haptics` — tactile feedback on iOS for a polished feel
- Do NOT install `@expo/vector-icons` — prefer `expo-image` with SF Symbols on native, or inline emoji/SVG for cross-platform

### Step 4: Configure Expo Router

Update `package.json` to set the entry point for expo-router:

```json
"main": "expo-router/entry"
```

Update `app.json` to add the scheme for deep linking:

```json
{
  "expo": {
    "scheme": "myapp",
    "web": {
      "bundler": "metro",
      "favicon": "./assets/favicon.png"
    }
  }
}
```

Update `tsconfig.json` to include path aliases:

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx", ".expo/types/**/*.ts", "expo-env.d.ts"]
}
```

### Step 5: Write the Sample App

Create the app using Expo Router with the `app/` directory structure. Follow **all** building-native-ui guidelines:

#### `app/_layout.tsx` — Root layout with Stack navigation

```tsx
import { Stack } from "expo-router/stack";

export default function RootLayout() {
  return (
    <Stack
      screenOptions={{
        headerStyle: { backgroundColor: "#0a0a0a" },
        headerTintColor: "#fff",
        headerTitleStyle: { fontWeight: "700" },
        contentStyle: { backgroundColor: "#0a0a0a" },
      }}
    />
  );
}
```

#### `app/index.tsx` — Main screen

Build a fun, beginner-friendly main screen. The code MUST follow these building-native-ui rules:

- **Inline styles** — do NOT use `StyleSheet.create` unless reusing across components
- **`Pressable`** instead of `TouchableOpacity`
- **`useWindowDimensions`** instead of `Dimensions.get()`
- **`ScrollView contentInsetAdjustmentBehavior="automatic"`** instead of `SafeAreaView` wrappers
- **`boxShadow`** CSS style instead of legacy React Native shadow/elevation props
- **`{ borderCurve: "continuous" }`** on rounded corners (not capsule shapes)
- **`{ fontVariant: ["tabular-nums"] }`** on counter numbers
- **Flex `gap`** instead of margin between items
- **`selectable` prop** on important `<Text>` elements
- **`expo-haptics`** on button presses (conditionally on iOS via `process.env.EXPO_OS`)
- **`Stack.Screen options={{ title: "..." }}`** for the page title, not custom text headers
- Use `LinearGradient` for colorful sections
- Include **comments explaining what each section does** in simple language
- Make it look polished and fun, not like a boring tutorial

The app should include:
- A gradient hero/banner section with the app name
- A tap counter with +/- and reset buttons (with haptic feedback)
- A grid of 4-6 fun emoji cards in a responsive layout
- A footer

**Keep it to just 2 files** (`_layout.tsx` + `index.tsx`). Beginners should see everything in one place. No extra component files unless the user asks.

### Step 6: Set Up Claude Code Config

Create `.claude/settings.json` with pre-approved permissions:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm install:*)",
      "Bash(npm run:*)",
      "Bash(npx expo:*)",
      "Bash(npx create-expo-app:*)",
      "Bash(mkdir:*)",
      "Bash(ls:*)",
      "Bash(cat:*)",
      "Bash(kill:*)",
      "Bash(lsof:*)",
      "Bash(curl:*)",
      "mcp__Claude_Preview__preview_start",
      "mcp__Claude_Preview__preview_stop",
      "mcp__Claude_Preview__preview_screenshot",
      "mcp__Claude_Preview__preview_snapshot",
      "mcp__Claude_Preview__preview_logs",
      "mcp__Claude_Preview__preview_console_logs",
      "mcp__Claude_Preview__preview_network",
      "mcp__Claude_Preview__preview_eval",
      "mcp__Claude_Preview__preview_resize",
      "mcp__Claude_Preview__preview_inspect",
      "mcp__Claude_Preview__preview_list",
      "WebSearch"
    ]
  }
}
```

Create `.claude/launch.json` for web preview.

**IMPORTANT:** Do NOT hardcode `--port` in runtimeArgs — use `autoPort: true` so the preview system can assign a free port automatically. Hardcoded ports cause conflicts when other processes (like tunnel mode) are running.

```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "expo-dev",
      "runtimeExecutable": "npx",
      "runtimeArgs": ["expo", "start", "--web"],
      "port": 8081,
      "autoPort": true
    }
  ]
}
```

### Step 7: Write CLAUDE.md

Create a `CLAUDE.md` in the project root:

```markdown
# My Expo App

## What is this?
This is a mobile app built with Expo and React Native using Expo Router for navigation.

## How to Run
- Ask Claude "show me my app" to see it in the web preview
- To test on your phone: ask Claude "start expo for my phone" — then scan the QR code with Expo Go app

## How to Build Things
Just ask Claude! Try things like:
- "Add a new screen that shows the current time"
- "Add tabs to the app"
- "Change the colors to blue and purple"
- "Add a settings page"

## UI Guidelines
This project follows the `expo:building-native-ui` skill. Key rules:
- Use Expo Router (`app/` directory) for all screens
- Inline styles, not StyleSheet.create
- Pressable, not TouchableOpacity
- useWindowDimensions, not Dimensions.get()
- ScrollView with contentInsetAdjustmentBehavior="automatic"
- boxShadow CSS style, not legacy shadow props
- borderCurve: "continuous" on rounded corners
- expo-haptics for tactile feedback on iOS
- Stack.Screen options for page titles

## Key Commands
- `npx expo start --web` — run in browser
- `npx expo start` — run for Expo Go on phone
- `npx expo start --tunnel` — run for phone over mobile internet

## Testing on Your Phone
1. Install "Expo Go" from App Store or Play Store
2. **Same Wi-Fi:** run `npx expo start`, scan QR code
3. **Different network:** run `npx expo start --tunnel`, use the tunnel URL

## Project Structure
- `app/_layout.tsx` — Root navigation layout
- `app/index.tsx` — Main screen
- `app.json` — App settings
- `assets/` — Images and icons
```

### Step 8: Start the Preview

1. Use `preview_start` with name `"expo-dev"` to launch the web server
2. Wait for the bundler to finish (check `preview_logs` for "Bundled" — first bundle can take 10-20s)
3. Take a `preview_screenshot` to show the user their running app
4. Also resize to mobile and take another screenshot

### Step 9: Welcome Message

After everything is set up, tell the user:

> **Your app is ready!** Here's what I set up:
>
> - Expo Router with native Stack navigation
> - A fun sample app with a counter and emoji cards
> - Haptic feedback, modern styling, and responsive layout
> - Web preview is running — you can see your app above
>
> **Try asking me things like:**
> - "Add a new screen with a form"
> - "Add tabs to the app"
> - "Make the emoji cards tappable and show details"
>
> **To test on your phone:** Ask me "start expo for my phone" and I'll show you how.

## Ongoing Development

**IMPORTANT:** When the user asks to modify or add UI after initial setup, always load the `expo:building-native-ui` skill first. It contains detailed references for:
- Animations (Reanimated)
- Native controls (Switch, Slider, DateTimePicker)
- Form sheets and modals
- Gradients
- Icons (SF Symbols via expo-image)
- Camera, audio, video
- Route structure conventions
- Search bars
- Storage (SQLite, AsyncStorage, SecureStore)
- Native tabs
- Toolbars and headers
- Visual effects (blur, liquid glass)
- WebGPU / Three.js
- Zoom transitions

## Phone Testing

When the user asks to test on their phone, **always pipe Metro output to a log file** so you can read errors, device connections, and bundle status directly.

### Step 1: Start Metro with logs (try localhost first)

```bash
# Kill any existing Expo process
kill $(lsof -ti :8081) 2>/dev/null

# Create the log file
export EXPO_LOG_FILE="/tmp/expo-metro-$(date +%s).log"

# Start Metro in localhost mode (default — simpler, faster, more reliable)
npx expo start > "$EXPO_LOG_FILE" 2>&1 &
EXPO_PID=$!

# Wait for Metro to be ready
for i in $(seq 1 20); do
  if grep -q "Metro waiting on" "$EXPO_LOG_FILE" 2>/dev/null; then
    echo "Metro is ready!"; break
  fi
  sleep 1
done

# Show the LAN URL
grep "Metro waiting on" "$EXPO_LOG_FILE"
```

Tell the user to open Expo Go on their phone (same Wi-Fi) and enter the `exp://192.168.x.x:8081` URL shown in the log.

### Step 2: If localhost doesn't work, switch to tunnel

Only use tunnel mode if the phone can't connect via localhost — different network, firewall issues, mobile data, corporate Wi-Fi blocking, etc.

```bash
# Stop the current Metro
kill $EXPO_PID 2>/dev/null
kill $(lsof -ti :8081) 2>/dev/null

# Restart with tunnel
export EXPO_LOG_FILE="/tmp/expo-metro-$(date +%s).log"
npx expo start --tunnel > "$EXPO_LOG_FILE" 2>&1 &
EXPO_PID=$!

# Wait for tunnel to be ready (takes longer, ~15-20s)
for i in $(seq 1 30); do
  if grep -q "Tunnel ready" "$EXPO_LOG_FILE" 2>/dev/null; then
    echo "Tunnel is ready!"; break
  fi
  sleep 1
done

# Get the tunnel URL
curl -s http://localhost:4040/api/tunnels | python3 -c "import sys,json; print(json.load(sys.stdin)['tunnels'][0]['public_url'])"
```

Give the user the URL as `exp://<tunnel-domain>` (replace `https://` with `exp://`).

### Monitoring logs (IMPORTANT)

The log file at `$EXPO_LOG_FILE` captures everything Metro outputs — device connections, JS errors, bundle status. **Read it proactively**, don't wait for the user to report problems:

```bash
# Check latest output
tail -40 "$EXPO_LOG_FILE"

# Check for errors
grep -i "error\|warn\|failed\|exception" "$EXPO_LOG_FILE"

# See device connections
grep -i "connected\|device\|iOS\|Android" "$EXPO_LOG_FILE"
```

**When to check the logs:**
- Right after the user says they opened the app
- After any code change — check if hot reload succeeded
- When the user reports something is wrong
- Periodically during active development

**If the log file variable is lost** (new shell), find the most recent one:
```bash
tail -40 "$(ls -t /tmp/expo-metro-*.log 2>/dev/null | head -1)"
```

### Stopping Metro

```bash
kill $EXPO_PID 2>/dev/null
```

**Note:** `npx expo start` in a non-interactive terminal (like Claude's Bash tool) cannot display QR codes. Always extract and provide the URL directly instead.

## Important Rules

- Follow `expo:building-native-ui` patterns for ALL UI code
- Routes go in `app/` directory — never co-locate components/utilities there
- Use kebab-case for file names (e.g. `comment-card.tsx`)
- After ANY code change, take a preview screenshot to show the result
- If something breaks, fix it and explain what went wrong in simple terms
- Suggest fun next steps after completing any task

## Troubleshooting

- **Port conflicts:** If port 8081 is in use, kill the process with `kill $(lsof -ti :8081)` before starting
- **Paths with spaces (e.g. iCloud):** These can cause issues with package resolution. If `npx expo start` fails with "Invalid package config", run `npm install` again and retry
- **zsh vs bash:** The default macOS shell is zsh. Do NOT use `shopt -s dotglob` (bash-only). Use `cp -a source/. dest/` to copy dotfiles instead
- **Tunnel not connecting:** Make sure `@expo/ngrok` is installed (`npm install @expo/ngrok`). If it still fails, try `npx expo start --tunnel` again
