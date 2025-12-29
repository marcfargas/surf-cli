# Pi Chrome Extension

Browser automation via Chrome extension with CLI and socket API.

## Features

- **CLI**: `pi-chrome` command for terminal-based browser control
- **Socket API**: JSON protocol via Unix socket for agent integration
- **50+ Tools**: Tabs, scrolling, input, screenshots, JavaScript execution
- **Page Understanding**: Accessibility tree extraction with element refs
- **CDP-based**: Bypasses CSP restrictions, works on any page

## Installation

```bash
npm install
npm run build
npm link              # Makes 'pi-chrome' CLI available globally
```

### Load Extension in Chrome

1. Open `chrome://extensions`
2. Enable "Developer mode"
3. Click "Load unpacked" and select the `dist/` folder

### Setup Native Host

```bash
cd native
./install.sh          # Install native messaging manifest
```

The extension auto-starts the socket server at `/tmp/pi-chrome.sock`.

## CLI Usage

```bash
pi-chrome <tool> [args] [options]
pi-chrome --help                    # Main help
pi-chrome <group>                   # Group help (tab, scroll, page, wait)
pi-chrome <tool> --help             # Tool help
pi-chrome --list                    # List all 50+ tools
```

### Tabs

```bash
pi-chrome tab.list
pi-chrome tab.new "https://google.com"
pi-chrome tab.switch 12345
pi-chrome tab.close 12345
```

### Navigation & Screenshots

```bash
pi-chrome navigate "https://example.com"
pi-chrome screenshot --output /tmp/shot.png
```

### Scrolling

```bash
pi-chrome scroll.top
pi-chrome scroll.bottom
pi-chrome scroll.info
pi-chrome scroll.to --ref "section_1"
```

### Input

```bash
pi-chrome click --ref "btn_1"
pi-chrome click --x 100 --y 200 --button double
pi-chrome type --text "hello"
pi-chrome smart_type --selector "#input" --text "hello" --submit
pi-chrome key Enter
pi-chrome key "cmd+a"

# Method flag: switch between CDP (real events) and JS (DOM manipulation)
pi-chrome type --text "hello" --selector "#input" --method js   # Uses smart_type
pi-chrome click --selector ".btn" --method js                   # Uses JS click()
```

### Page Inspection

```bash
pi-chrome page.read                 # Accessibility tree
pi-chrome page.text                 # Extract all text
pi-chrome page.state                # Modals, loading state
```

### Waiting

```bash
pi-chrome wait 2                    # Wait 2 seconds
pi-chrome wait.element --selector ".loaded"
pi-chrome wait.network              # Wait for network idle
pi-chrome wait.url --pattern "*/success*"
```

### JavaScript

```bash
pi-chrome js "return document.title"
pi-chrome js "return document.querySelector('#btn').textContent"
```

### Options

```bash
--tab-id <id>         Target specific tab
--json                Output raw JSON response
```

## Socket API

Send JSON to `/tmp/pi-chrome.sock`:

```bash
echo '{"type":"tool_request","method":"execute_tool","params":{"tool":"tab.list","args":{}},"id":"1"}' | nc -U /tmp/pi-chrome.sock
```

Response:
```json
{"type":"tool_response","id":"1","result":{"content":[{"type":"text","text":"..."}]}}
```

## Available Tools

### Dot-notation (preferred)

| Group | Tools |
|-------|-------|
| `tab.*` | `list`, `new`, `switch`, `close` |
| `scroll.*` | `top`, `bottom`, `to`, `info` |
| `page.*` | `read`, `text`, `state` |
| `wait.*` | `element`, `network`, `url` |

### Core Tools

| Tool | Description |
|------|-------------|
| `screenshot` | Capture screenshot |
| `navigate` | Go to URL |
| `js` | Execute JavaScript (use `return` for values) |
| `click` | Click by ref or coordinates |
| `type` | Type text |
| `smart_type` | Type into selector with contenteditable support |
| `key` | Press key (Enter, Escape, cmd+a) |
| `hover` | Hover over element |
| `drag` | Drag between points |
| `wait` | Wait N seconds |

### Legacy Tools (still supported)

| Tool | Description |
|------|-------------|
| `list_tabs`, `new_tab`, `switch_tab`, `close_tab` | Tab management |
| `scroll_to_position`, `get_scroll_info` | Scrolling |
| `read_page`, `get_page_text`, `page_state` | Page inspection |
| `wait_for_element`, `wait_for_network_idle`, `wait_for_url` | Waiting |
| `javascript_tool` | JS execution (alias: `js`) |
| `computer` | Anthropic computer-use format wrapper |
| `read_console_messages`, `read_network_requests` | Dev tools |

Run `pi-chrome --list` for all 50+ tools.

## Architecture

```
CLI (pi-chrome) ─────► Socket (/tmp/pi-chrome.sock) ─────► host.cjs ─────► Extension ─────► CDP
```

## Limitations

- Cannot automate `chrome://` pages, Web Store, or other extensions
- First CDP operation on a new tab takes ~5-8s (debugger attachment)
- Shows "Chrome is being controlled" banner when CDP active

## Development

```bash
npm run dev       # Watch mode
npm run build     # Production build
npm run check     # Type check
```

### After code changes

- **Extension changes** (`src/`): Reload extension in `chrome://extensions`
- **Host changes** (`native/host.cjs`): Kill existing process or reload extension
  ```bash
  pkill -f host.cjs
  ```

### Debugging

Service worker logs: `chrome://extensions` > Pi Agent > "Inspect views: service worker"
