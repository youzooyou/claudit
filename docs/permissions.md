# Claudit Permission Justification

**Purpose:** This document explains why Claudit requests each Chrome extension permission. It is intended for:
1. Chrome Web Store reviewers during submission
2. Users who want to audit the extension before install
3. Project maintainers adding new features that might require new permissions

**Manifest version:** 3
**Extension version:** 1.0.0
**Last updated:** 2026-04-16

---

## Summary table

All permissions are declared in `public/manifest.json` (auto-copied to `dist/manifest.json` by Vite).

| Permission | Category | Used for | Alternatives considered |
|------------|----------|----------|------------------------|
| `activeTab` | Standard | Targeting the tab the user is currently viewing | None — required for any per-tab action |
| `scripting` | Standard | Injecting read-only DOM inspection functions | Content scripts (rejected: stale on SPA nav) |
| `storage` | Standard | Persisting API key, project config, test results | `localStorage` (rejected: not available to SW) |
| `alarms` | Standard | Keeping SW alive during long agent sessions | `setInterval` (rejected: killed on SW hibernation) |
| `tabs` | Standard | Reading tab URL/title, coordinating across tabs | Content script messaging (rejected: incomplete) |
| `webRequest` | **Sensitive** | Capturing XHR/fetch API requests made by the target page for test verification. Request bodies are not captured. | DevTools Protocol (used in parallel, not a replacement) |
| `sidePanel` | Standard | Primary UI is a Chrome side panel | Popup (rejected: too small), content script overlay (rejected: DOM pollution) |
| `windows` | Standard | Focusing the correct browser window for side panel | None — required by `chrome.sidePanel.open()` |
| `webNavigation` | Standard | Detecting tab URL changes and navigation events | `tabs.onUpdated` (insufficient for SPA nav detection) |
| `debugger` | **Sensitive** | CDP-based click, keyboard input, and screenshots | Content script events (rejected: unreliable on React admins) |
| `offscreen` | Standard | Running File System Access API in a DOM context | SW direct access (rejected: no `window`/`document`) |
| `notifications` | Standard | Alerting the user when a long test completes | None — user is often on another tab during runs |
| `<all_urls>` host | **Sensitive — Optional** | Users run tests against arbitrary admin URLs; domains are user-selected at runtime and cannot be enumerated. Declared as `optional_host_permissions` — granted on demand only. | Per-domain prompts (disrupts UX flow) |
| `declarativeNetRequestWithHostAccess` | **Sensitive — Optional** | Routes requests to a user-configured custom Base URL / corporate proxy endpoint. Declared as `optional_permissions` — only active when a non-default Base URL is configured. | Hardcoded Anthropic endpoint (rejected: corporate deployments need custom endpoints) |

---

## Detailed justification

### `activeTab`

**Use:** Identifies the tab the user is currently interacting with so that agent tool calls target the right page.

**Why needed:** Without `activeTab`, the extension cannot reliably pair a user's chat message with the DOM they expect the agent to operate on. All automation tools use `chrome.tabs.get(tabId)` to confirm the target before executing.

**Data accessed:** Tab ID, URL, and title of the active tab only when the side panel is open.

---

### `scripting`

**Use:** Injects small read-only DOM inspection functions into the target page to capture selectors, form state, and grid contents.

**Why needed:** Modern admin UIs (React, Vue, AgGrid) use synthetic events and virtual DOM. A content script running in an isolated world can read structure but not reliably observe React state. `chrome.scripting.executeScript` with `world: 'MAIN'` is the only way to inspect React fiber nodes for accurate test verification.

**Data accessed:** Read-only DOM queries. Claudit never modifies page state through injected scripts — modification is handled via CDP (`debugger`).

**Example:** `inspect_page` tool injects a function that walks the DOM and returns a compact snapshot of visible inputs, buttons, and grids. The snapshot is sent to Claude as context, never persisted.

---

### `storage`

**Use:** Persists:
- Claude API credentials (API key or bearer token) + user-configurable `Base URL` (`chrome.storage.local`)
- Project configuration: specs root path, data directory path, user preferences (`chrome.storage.local`)
- Data processing consent flag (`chrome.storage.local`)
- Recent test results for quick review (`chrome.storage.local`)
- Agent runtime state for Service Worker hibernation recovery (`chrome.storage.session`)

**Why needed:** Service Workers cannot use `localStorage`. `chrome.storage.local` is the standard persistence mechanism. Without it, users would need to re-enter their API key on every browser restart.

**Data stored:** Only data the user explicitly provides or generates during tests. Never transmitted off-device.

---

### `alarms`

**Use:** Creates a 30-second periodic alarm to keep the Service Worker alive during long-running agent sessions.

**Why needed:** Chrome MV3 Service Workers are terminated after ~30 seconds of inactivity, which would interrupt an ongoing Claude API streaming response or multi-step agent loop. `chrome.alarms.create` is the Chrome-recommended keepalive mechanism.

**Data accessed:** None. The alarm handler is a no-op that exists only to prevent hibernation.

---

### `tabs`

**Use:** Reading the URL and title of tabs other than the active one for multi-tab coordination (e.g., sidebar state per tab), and detecting tab removal/movement to clean up session state.

**Why needed:** Claudit maintains per-tab session state (chat history, agent state) keyed by tab ID. When a user closes or moves a tab, the extension must archive or discard the associated session, which requires `chrome.tabs.onRemoved`, `chrome.tabs.onDetached`, and `chrome.tabs.onAttached` listeners.

**Data accessed:** Tab URL, title, and ID for tabs in windows where the side panel has been opened.

---

### `webNavigation`

**Use:** Detects URL changes within the same tab (e.g., SPA navigation from `/admin/orders` to `/admin/items`) to update the sidebar's context and clear stale network captures.

**Why needed:** `chrome.tabs.onUpdated` fires on every load event but does not reliably distinguish SPA navigations. `webNavigation` provides `onCompleted` and `onHistoryStateUpdated` events that cover both full-page loads and pushState navigation.

**Data accessed:** Tab ID and URL of navigation events within open tabs. No content is read.

---

### `webRequest`

**Use:** Captures XHR/fetch API requests made by the target admin page for use in test verification (e.g., confirming that clicking "Save" triggered the expected `POST /admin/items` call).

**Why needed:** E2E tests of web admins frequently need to assert that UI actions produced the correct API calls. `chrome.webRequest.onBeforeRequest` is used to intercept and log these requests.

**Data accessed:**
- URL, HTTP method, status code, and timing of XHR/fetch requests from the tab under test only.
- Request bodies are **not** captured (the `requestBody` optional info spec is not used).
- Static resources (images, fonts, scripts, stylesheets) are excluded by type filter.
- Requests from other tabs are excluded by tab ID filter.
- Claude API traffic from the extension itself (`api.anthropic.com`) is explicitly excluded.
- Capture is opt-in per test session (started by `browser_network_start` tool call). Entries are kept in a 500-entry ring buffer in `chrome.storage.session` only; never sent anywhere except to Claude when the user explicitly retrieves them via `browser_network_get` during a chat.

---

### `sidePanel`

**Use:** Claudit's primary user interface is a Chrome side panel rather than a popup or content-script overlay.

**Why needed:** The panel hosts the chat UI, result viewer, and test controls. A popup would be too small (closes on blur), and a content script overlay would pollute every page's DOM. The side panel is the correct Chrome primitive for persistent auxiliary UI.

---

### `windows`

**Use:** Calling `chrome.sidePanel.open({ windowId })` requires a window ID. The extension reads the last-focused window to open the panel when the user presses the Alt+Shift+S shortcut.

**Why needed:** `chrome.sidePanel.open` does not accept "current window" — it requires an explicit window ID, which requires the `windows` permission to query.

**Data accessed:** Window IDs only. No content.

---

### `debugger`

**Use:** Attaches the Chrome DevTools Protocol to the target tab to perform:
- Mouse click dispatch at pixel coordinates
- Keyboard input dispatch
- Full-page and element screenshots
- Setting device viewport metrics for non-active tabs during multi-tab test scenarios

**Why needed:** Content-script-dispatched events (`element.click()`, `element.dispatchEvent(new KeyboardEvent())`) are unreliable on React admins because React's synthetic event system rewrites the event handler chain. CDP dispatches events at the browser level, below React, making them indistinguishable from real user input. This is essential for testing interactive UIs.

**Important:** Chrome shows a persistent yellow "Claudit started debugging this browser" banner whenever the debugger is attached. Users always know when CDP is active. Claudit attaches and detaches within each tool call via a `try/finally` block to minimize banner duration. Tabs that already have DevTools open are skipped to avoid conflicts.

**Data accessed:** Only the tab the user is testing, only during explicit tool calls.

---

### `offscreen`

**Use:** Creates a single Offscreen Document (`offscreen.html`) that holds `FileSystemDirectoryHandle` objects for user-selected folders. The Service Worker cannot use the File System Access API directly because it has no `window`/`document` context.

**Why needed:** Claudit writes test results and reads specification files from user-selected folders. The File System Access API requires a DOM context for handle resurrection from IndexedDB. An Offscreen Document is the Chrome-recommended pattern for this.

**Data accessed:** Only files in directories the user explicitly selected via the directory picker.

---

### `notifications`

**Use:** Displays a Chrome system notification when a long-running test session completes, so users who have switched to another tab or application know to return.

**Why needed:** Test runs can take several minutes. Without a notification, users must poll the side panel manually. The notification contains only the test title and pass/fail count — no sensitive data.

**Data accessed:** None beyond the test result summary the user is actively viewing.

---

### `<all_urls>` host permission — **Optional**

**Declaration:** `optional_host_permissions` in manifest.json. This permission is **not granted at install time** — it is requested only when the user opens the Claudit side panel on a target page.

**Use:** Allows content-script injection, CDP attachment, and `webRequest` capture on any URL the user visits during a test session.

**Why needed:** Claudit is a tool for testing arbitrary web admin panels. The target URLs are not known in advance — a QA engineer might test `admin.example.com` on Monday and `staging.another.com` on Tuesday. Unlike a single-site extension, Claudit has no way to enumerate required hosts at install time.

**Why optional (not required):** Declaring `<all_urls>` as `optional_host_permissions` means users must explicitly grant broad host access when they first open the side panel. This provides a clear consent moment and allows Chrome to show a per-permission prompt, giving users full awareness before access is granted.

**Why sensitive and how risk is mitigated:**
1. Never activates automation without an explicit user chat message
2. Clear visual indicators (side panel open, debugger banner) whenever the extension is acting on a page
3. All code is published and auditable at https://github.com/youzooyou/claudit
4. No data is sent anywhere except the Claude API endpoint the user configured in Options and the user's own selected folders

---

### `declarativeNetRequestWithHostAccess` — **Optional**

**Declaration:** `optional_permissions` in manifest.json. This permission is **not granted at install time**.

**Use:** Enables declarative network request modification to route API calls to a user-configured custom `Base URL` (e.g., a corporate proxy or private Claude gateway) rather than the default `https://api.anthropic.com`.

**Why needed:** Some organizations require all Claude API traffic to pass through an internal proxy or gateway (e.g., for quota management, compliance logging, or regional data residency). Without this permission, the extension cannot modify outbound request headers to reach a non-Anthropic endpoint with proper authentication.

**Why optional:** This permission is only needed when the user sets a non-default `Base URL` in the Options page. Users who connect directly to Anthropic's public API never trigger this permission. Keeping it optional limits the surface area for users who don't need it.

**Data accessed:** Only the `x-api-key` or `Authorization` header value the user provided. Applied exclusively to the domain matching the `Base URL` the user configured. No other requests are modified.

---

## Chrome Web Store submission text

When filling out the Web Store developer console "Permission justification" fields, use the following short texts (each under the ~1000 character limit):

### Single purpose

> Claudit is a browser automation testing tool that lets software QA engineers drive web admin panels via Claude's tool-use API. The extension captures screenshots, inspects DOM, clicks buttons, and verifies network requests during user-initiated test sessions, then records the results locally.

### `activeTab` justification

> Required to target the tab the user is currently viewing during a test session. Used to fetch the URL and ID of the active tab and coordinate tool calls against the correct page.

### `scripting` justification

> Required to inject read-only DOM inspection functions into the target admin page. Used to capture form state, grid contents, and ARIA selectors that the agent needs as context for test verification. No page state is modified via scripting.

### `storage` justification

> Required to persist the user's Claude API credentials (API key or bearer token), the user-configurable API Base URL, project configuration (spec folder, data folder), data consent flag, and test result history. Service Workers cannot use localStorage, so chrome.storage.local is the only option.

### `alarms` justification

> Required to keep the Service Worker alive during long Claude API streaming responses. Without a periodic alarm, Chrome terminates the SW after 30 seconds of inactivity, interrupting agent sessions.

### `tabs` justification

> Required to read URL and title of tabs, and to clean up per-tab session state when tabs close or are moved between windows. Claudit maintains separate chat history per tab.

### `webNavigation` justification

> Required to detect SPA navigation within the same tab (pushState/replaceState). chrome.tabs.onUpdated does not reliably cover in-page navigation on React/Vue admin panels, which is needed to update the sidebar context and clear stale network captures.

### `webRequest` justification

> Required to capture XHR/fetch API requests made by the admin page for test verification. E2E tests frequently assert that UI actions trigger expected API calls. Only XHR/fetch requests from the tab under test are captured; request bodies, static resources, and other tabs' traffic are excluded. Capture is opt-in per session and stored only in session storage.

### `sidePanel` justification

> Claudit's primary UI is a Chrome side panel (chat + results + controls). A popup would be too small and a content-script overlay would pollute every page's DOM.

### `windows` justification

> Required by `chrome.sidePanel.open({ windowId })`, which needs an explicit window ID from `chrome.windows.getLastFocused`.

### `debugger` justification

> Required to dispatch click, keyboard, and screenshot operations via Chrome DevTools Protocol. Content-script events are unreliable on React admins because React's synthetic event system intercepts them. CDP dispatches at the browser level, making automation reliable. Chrome's yellow debugger banner keeps users informed whenever CDP is attached.

### `offscreen` justification

> Required to run File System Access API in a DOM context. Service Workers have no window/document, so an Offscreen Document holds the FileSystemDirectoryHandle objects for user-selected folders.

### `notifications` justification

> Required to alert the user when a long-running test session completes. Test runs can take several minutes; users often switch tabs during runs and need to know when results are ready.

### `<all_urls>` host permission justification

> Claudit is a QA testing tool for arbitrary web admin panels. Target URLs are not known in advance — QA engineers test different staging and production environments. Declared as optional_host_permissions so users explicitly grant access when opening the panel, not at install time. All automation requires user initiation via chat, and Chrome's debugger banner makes activity visible.

### `declarativeNetRequestWithHostAccess` justification

> Required only when the user configures a custom Base URL (corporate proxy or private Claude gateway) in the Options page. Modifies the Authorization header to route traffic correctly. Declared as optional — users connecting to the default Anthropic API endpoint are never prompted for this permission.

---

## Adding new permissions

Before adding a new permission to `public/manifest.json`:

1. Add a detailed justification section to this document
2. Confirm no existing permission covers the use case
3. Update the Privacy Policy if the new permission accesses new data categories
4. Add the short Web Store justification text above
5. Test the build with `npm run build` and verify `dist/manifest.json` is correct
