# Claudit — Public Documentation

**Claude-Led Automated UI-Driven Inspection & Testing** — an AI QA agent for web applications, built as a Chrome extension.

> This repository hosts **public-facing documentation** for the Claudit Chrome extension: Privacy Policy, permission justification, and user-oriented guides. The extension's source code is maintained in a separate repository and is not currently open source.

## 🎯 What Claudit is

Claudit lets QA engineers, testers, and developers drive their own web admin panels via Claude's tool-use API. It takes screenshots, reads DOM structure, clicks buttons, fills forms, observes network requests, and compares observed behavior against specifications you provide — all inside your browser, with no remote orchestration server.

**Intended use:** quality-assurance (QA) tool for **end-to-end (E2E) testing** of web applications you own, operate, or are explicitly authorized to test.

**Typical workflow:**
1. Open a staging or production admin panel you are authorized to test
2. Load a specification file (`spec.yaml`) describing expected behavior
3. Ask Claudit to run a regression test or explore an edge case
4. Claudit drives the page and reports pass/fail findings
5. Results are written to a folder you selected, for later review or CI integration

**Scope:**
- ✅ Regression testing, edge-case exploration, spec compliance checking
- ✅ Capturing reproducible test evidence (screenshots, network traces) for bug reports
- ❌ Scraping third-party sites you do not own
- ❌ Circumventing authentication, rate limits, or terms of service
- ❌ Unauthorized automation on a user's personal accounts

## 🔐 Privacy at a glance

- **No data collection.** Claudit has no servers of its own.
- **No tracking.** No analytics, no telemetry, no crash reporting, no ads.
- **You control the API endpoint.** Default is Anthropic's public Claude API; you can point `Base URL` at any Claude-compatible endpoint (corporate proxy, self-hosted gateway, regional variant).
- **You control authentication.** Either an Anthropic API key (`x-api-key`) or a bearer token (`Authorization: Bearer`).
- **All credentials and files stay on your device** unless you explicitly send a chat to the endpoint you configured.
- **No automation without user action.** Every test session is initiated by an explicit user chat message, and Chrome's yellow "debugger attached" banner makes automation visible throughout.

Full Privacy Policy: [`PRIVACY_POLICY.md`](./PRIVACY_POLICY.md) (English + Korean)

Terms of Service: [`TERMS_OF_SERVICE.md`](./TERMS_OF_SERVICE.md) (English + Korean)

## 🛡️ Permissions

Claudit requests several Chrome extension permissions. Each is used strictly for the stated purpose.

Full justification: [`docs/permissions.md`](./docs/permissions.md)

Summary:
- `debugger` — CDP-based click/input/screenshot (reliable on React admins); Chrome's yellow banner is always shown when attached
- `scripting` — inject read-only DOM inspection code (no page modification)
- `webRequest` — capture XHR/fetch requests from the test tab only for verification; request bodies are not captured
- `webNavigation` — detect SPA navigation (pushState/replaceState) within a tab to update sidebar context and clear stale captures
- `storage`, `alarms`, `tabs`, `sidePanel`, `windows` — standard extension APIs
- `offscreen` — required for File System Access API (DOM context)
- `notifications` — alert users when long-running tests complete
- `declarativeNetRequestWithHostAccess` *(optional — not granted at install)* — only activated when a custom Base URL (corporate proxy) is configured in Options

## 📦 Installation

_Chrome Web Store listing: coming soon._

## 🤝 Contact

This is a documentation-only repository. Questions about privacy, permissions, or the extension's behavior are welcome as issues:

- **Issues:** https://github.com/youzooyou/claudit/issues

For private matters (e.g., security disclosure), open a minimal public issue requesting a private contact method.

## 📄 License

Documentation in this repository is published under the [MIT License](./LICENSE). The Claudit Chrome extension itself is **not open source**; its source code is maintained privately.
