# Pi Chrome Native Messaging Host

Native messaging host that bridges pi-coding-agent to the Chrome extension via Unix socket.

## Architecture

```
Pi-Agent → Unix Socket (/tmp/pi-chrome.sock) → Native Host (host.cjs) → Chrome Native Messaging → Extension
```

## Files

| File | Purpose |
|------|---------|
| `host.cjs` | Main native host with socket server and tool request handling |
| `cli.cjs` | CLI tool for direct browser automation |
| `protocol.cjs` | Chrome native messaging protocol helpers |
| `host-wrapper.py` | Python wrapper for native host execution |
| `host.sh` | Shell script to start the host |

## Setup

1. Install the native host manifest:
```bash
mkdir -p ~/Library/Application\ Support/Google/Chrome/NativeMessagingHosts
cat > ~/Library/Application\ Support/Google/Chrome/NativeMessagingHosts/com.anthropic.pi_chrome.json << EOF
{
  "name": "com.anthropic.pi_chrome",
  "description": "Pi Chrome Extension Native Host",
  "path": "$PWD/host-wrapper.py",
  "type": "stdio",
  "allowed_origins": ["chrome-extension://YOUR_EXTENSION_ID/"]
}
EOF
```

2. Start the native host:
```bash
node host.cjs
```

The host creates a Unix socket at `/tmp/pi-chrome.sock`.

## CLI Usage

```bash
pi-chrome <command> [args] [options]
```

### Common Commands

| Command | Description |
|---------|-------------|
| `navigate <url>` | Go to URL (alias: `go`) |
| `click <ref>` | Click element by ref or coordinates |
| `type <text>` | Type text at cursor or into element |
| `screenshot` | Capture screenshot (alias: `snap`) |
| `page.read` | Get page accessibility tree (alias: `read`) |
| `search <term>` | Search for text in page (alias: `find`) |

### Navigation

```bash
pi-chrome go "https://example.com"
pi-chrome back
pi-chrome forward
pi-chrome tab.reload --hard
```

### Page Interaction

```bash
pi-chrome read                           # Get interactive elements
pi-chrome click e5                       # Click by element ref
pi-chrome click --selector ".btn"        # Click by CSS selector
pi-chrome click 100 200                  # Click by coordinates
pi-chrome type "hello" --submit          # Type and press Enter
pi-chrome key Escape                     # Press key
```

### Screenshots

```bash
pi-chrome screenshot --output /tmp/shot.png
pi-chrome screenshot --annotate --output /tmp/labeled.png
pi-chrome screenshot --fullpage --output /tmp/full.png
pi-chrome snap                           # Auto-saves to /tmp
```

### Tabs

```bash
pi-chrome tab.list
pi-chrome tab.new "https://example.com"
pi-chrome tab.switch 123
pi-chrome tab.close 123
pi-chrome tab.group --name "Work" --color blue
```

### Cookies

```bash
pi-chrome cookie.list
pi-chrome cookie.get --name "session"
pi-chrome cookie.set --name "foo" --value "bar"
pi-chrome cookie.clear --all
```

### Bookmarks & History

```bash
pi-chrome bookmark.add
pi-chrome bookmark.list --limit 20
pi-chrome history.list --limit 10
pi-chrome history.search "github"
```

### Other

```bash
pi-chrome zoom 1.5                       # Set zoom to 150%
pi-chrome resize --width 1280 --height 720
pi-chrome wait 2                         # Wait 2 seconds
pi-chrome js "return document.title"     # Execute JavaScript
```

### Help

```bash
pi-chrome --help                         # Basic help
pi-chrome --help-full                    # All commands
pi-chrome <command> --help               # Command details
pi-chrome --find <query>                 # Search commands
pi-chrome --about refs                   # Topic guide
```

## Protocol

### Tool Request

```json
{
  "type": "tool_request",
  "method": "execute_tool",
  "params": {
    "tool": "TOOL_NAME",
    "args": { ... },
    "tabId": 123
  },
  "id": "unique-request-id"
}
```

### Tool Response (Success)

```json
{
  "type": "tool_response",
  "id": "unique-request-id",
  "result": {
    "content": [
      { "type": "text", "text": "Result message" }
    ]
  }
}
```

### Tool Response (With Image)

```json
{
  "type": "tool_response",
  "id": "unique-request-id",
  "result": {
    "content": [
      { "type": "text", "text": "Screenshot captured" },
      { "type": "image", "data": "base64...", "mimeType": "image/png" }
    ]
  }
}
```

### Tool Response (Error)

```json
{
  "type": "tool_response",
  "id": "unique-request-id",
  "error": {
    "content": [{ "type": "text", "text": "Error message" }]
  }
}
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Socket not found | Ensure `node host.cjs` is running |
| No response | Check extension is loaded in Chrome |
| "Content script not loaded" | Navigate to page first |
| Slow first operation | Normal - CDP debugger attachment takes ~5s |
