# 🔍 DevTools Inspector — AI Agent Bridge

Connect an AI agent directly to a live Chrome instance via the **Chrome DevTools Protocol (CDP)**. The AI gets real-time access to everything you see in DevTools: network traffic, DOM, console logs, cookies, localStorage, and more.

## Architecture

```
Chrome (--remote-debugging-port=9222)
        │  CDP WebSocket
        ▼
  devtools-bridge  (Node.js)
  ├── CDP client   ← subscribes to Network, DOM, Console, Page, Runtime
  ├── Ring buffer  ← stores last 500 events per category
  ├── REST API     ← http://localhost:3000  ← AI agent queries here
  ├── SSE stream   ← real-time event feed
  └── Dashboard    ← http://localhost:3000  ← visual UI
        │
        ▼
   Claude AI Agent  (npm run agent)
```

## Quick Start

### 1. Install dependencies
```bash
npm install
```

### 2. Set up your API key
```bash
copy .env.example .env
# Edit .env and add your ANTHROPIC_API_KEY
```

### 3. Launch Chrome with remote debugging
```bash
npm run launch-chrome
# Or with your real profile (gives access to your logged-in sessions):
npm run launch-chrome -- --real
# Or with a specific URL:
npm run launch-chrome -- --url https://example.com
```

### 4. Start the bridge server
```bash
npm start
# Dashboard: http://localhost:3000
```

### 5. Run the AI agent
```bash
# Interactive mode:
npm run agent

# One-shot mode:
npm run agent -- "What API calls did this page make?"
npm run agent -- "Are there any console errors?"
npm run agent -- "What auth tokens are stored in cookies?"
```

## API Reference

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/status` | Bridge health + current page info |
| `GET` | `/tabs` | List all open Chrome tabs |
| `POST` | `/tabs/:id/attach` | Switch to a specific tab |
| `POST` | `/connect` | (Re)connect to Chrome |
| `GET` | `/network` | All captured network requests |
| `GET` | `/network?url=api&method=POST&status=404` | Filtered requests |
| `GET` | `/network/failed` | Failed network requests |
| `GET` | `/network/:id` | Single request with full headers + body |
| `GET` | `/console` | Console log entries |
| `GET` | `/console?level=error` | Filtered by level |
| `GET` | `/dom` | Full live DOM as HTML |
| `GET` | `/dom?format=json` | DOM as JSON with length |
| `GET` | `/dom/query?selector=button.submit` | CSS selector query |
| `GET` | `/page` | URL, title, cookies, page events |
| `POST` | `/eval` | Run JS in the page: `{"expression": "document.title"}` |
| `GET` | `/storage` | localStorage + sessionStorage |
| `GET` | `/cookies` | All cookies |
| `GET` | `/events/stream` | SSE real-time event stream |
| `POST` | `/clear` | Clear buffer |

## Agent Examples

```bash
# Debug failing API calls
npm run agent -- "Which API calls returned errors, and what did the response bodies say?"

# Authentication debugging
npm run agent -- "Is the user logged in? Check cookies and localStorage for auth tokens."

# Page analysis
npm run agent -- "What third-party scripts is this page loading?"

# Form debugging
npm run agent -- "Find all form elements on the page and check if they have validation"

# Performance
npm run agent -- "List all requests that took more than 1 second"
```

## Configuration

Edit `.env` to configure:

```env
CHROME_HOST=localhost          # Chrome debugging host
CHROME_PORT=9222               # Chrome debugging port
API_PORT=3000                  # Bridge API port
ANTHROPIC_API_KEY=sk-ant-...   # Your Anthropic API key
MAX_BODY_SIZE=1048576          # Max response body to capture (1MB)
BUFFER_SIZE=500                # Max events per category in memory
```

## Manual Chrome Launch (without the helper)

```powershell
# Windows
& "C:\Program Files\Google\Chrome\Application\chrome.exe" `
  --remote-debugging-port=9222 `
  --user-data-dir="$PWD\chrome-profile"
```

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=./chrome-profile
```

> **Note:** Microsoft Edge also supports CDP — just use `msedge.exe` instead of `chrome.exe`.

## SSE Event Types

Subscribe to `/events/stream` to receive real-time events:

| Event type | Description |
|------------|-------------|
| `network:request` | New request started |
| `network:response` | Response received |
| `network:body` | Response body available |
| `network:failed` | Request failed |
| `network:done` | Request finished loading |
| `ws:created` | WebSocket connection opened |
| `ws:frame` | WebSocket frame sent/received |
| `console:entry` | Console log/warn/error |
| `console:exception` | Uncaught exception |
| `page:navigate` | Page navigation |
| `page:load` | Load event fired |
| `bridge:connected` | CDP connected |
| `bridge:disconnected` | CDP disconnected |
