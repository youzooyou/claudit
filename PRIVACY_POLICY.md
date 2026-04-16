# Claudit Privacy Policy

**Last updated:** 2026-04-16
**Applies to:** Claudit Chrome Extension (version 1.0.0 and later)

---

## 1. Summary

Claudit is a local browser automation testing tool. It runs entirely inside your browser and communicates **only** with the Claude API endpoint you configure in the Options page when you explicitly start a chat or test session. We do not operate servers, we do not collect analytics, and we do not track you.

**Short version:**
- 🟢 All data stays on your device except prompts you send to the Claude endpoint you configured
- 🟢 You choose the endpoint: Anthropic's public API, a corporate proxy, or a self-hosted gateway
- 🟢 Before any data is transmitted, you must explicitly agree to a data processing disclosure shown inside the extension
- 🟢 No analytics, no telemetry, no crash reporting
- 🟢 No advertising, no third-party SDKs
- 🟢 You control your credentials and file permissions at all times

---

## 2. What Claudit does (Intended Use)

**Claudit is a quality-assurance (QA) tool for end-to-end (E2E) testing of web applications.** It is built for software testers, QA engineers, and developers who need to verify that their own web admin panels and internal tools behave correctly against documented specifications.

A typical workflow:
1. A QA engineer opens a staging or production admin panel they are authorized to test
2. They load a specification file (`spec.yaml`) describing the expected behavior of a screen
3. They ask Claudit to run a regression test case or explore an edge case
4. Claudit drives the page (click, type, screenshot, observe network requests), compares the observed behavior against the spec, and reports pass/fail findings
5. Results are written to a folder the user selected, for later review

**Scope of intended use:**
- ✅ Testing web applications you own, operate, or are explicitly authorized to test
- ✅ Regression testing, edge-case exploration, spec compliance checking
- ✅ Capturing reproducible test evidence (screenshots, network traces) for bug reports
- ✅ Running test plans as part of an internal QA process

**Out of scope (not designed for, and not supported):**
- ❌ Scraping content from third-party sites you do not own
- ❌ Circumventing authentication, rate limits, or terms of service of other websites
- ❌ Automating actions on a user's personal accounts on services they did not build
- ❌ Any form of surveillance, credential theft, or unauthorized data extraction

All automation runs inside your browser. There is no remote orchestration server. Claudit has no ability to operate unattended — every test session is initiated by an explicit user chat message, and Chrome's yellow "debugger attached" banner makes automation visible to the user throughout.

---

## 3. Data we collect

**None.** Claudit does not collect, transmit, or store any data on servers we operate, because we do not operate any servers.

---

## 4. User consent

Before Claudit transmits any data to a Claude API endpoint, you must explicitly acknowledge a data processing disclosure shown in the Options page. This disclosure lists exactly what types of data may be sent during a test session, and you must check a consent checkbox to enable the extension's core functionality.

You can withdraw consent at any time by unchecking that box in the Options page. Withdrawing consent does not delete locally stored data.

---

## 5. Data sent to the Claude API endpoint you configured

### Where your data goes

Claudit sends request payloads to **the Claude API endpoint configured in the Options page**. This endpoint is user-controlled:

- **Default:** `https://api.anthropic.com` (Anthropic's public Claude API)
- **Custom:** any Claude-compatible endpoint the user specifies in the `Base URL` field, for example:
  - A corporate proxy that adds organization logging, quota, or compliance controls
  - A self-hosted or private gateway in front of Anthropic
  - A regional/government variant with separate data residency guarantees

**Claudit does not choose the endpoint and does not validate its identity.** If you change `Base URL`, all subsequent requests go to that URL instead — Claudit will not warn you, log the change, or route around it. The responsibility for understanding the privacy, logging, and retention practices of the endpoint you point at belongs to you or your organization.

**What we send on each request:**

| Data | When | Purpose |
|------|------|---------|
| Your chat messages | Every turn | Agent prompt |
| Screenshots of the current tab | Tool call (`browser_capture`) | Visual context for the model |
| DOM snapshots (HTML structure, form state, grid contents) | Tool call (`inspect_page`, `get_form_state`) | Page understanding — passwords are excluded |
| Network request URLs from the active test tab | Tool call (`browser_network_get`) | API verification — request bodies and other tabs' traffic are not captured |
| Specification files you load | Session start | Test planning |
| URL of the active tab | Every turn | Context identification |

> **Note on network capture:** Only XHR/fetch (API) requests from the tab you are currently testing are recorded. Static resources (images, fonts, scripts, stylesheets) and requests from other tabs are excluded. Request bodies are not captured. Claude API traffic from the extension itself is also excluded.

**Authentication:** One of the following credentials is read from local storage and attached to each request:
- **`x-api-key`** header — your Anthropic API key (standard), *or*
- **`Authorization: Bearer <token>`** header — a bearer token, typically used for proxies or gateways that require token-based auth

These credentials are never transmitted anywhere except the `Base URL` you configured. If the default `https://api.anthropic.com` is used, the relevant governing policy is [Anthropic's Privacy Policy](https://www.anthropic.com/legal/privacy). If you use a custom endpoint, the governing policy is whatever agreement you have with that endpoint's operator.

### No other third parties

Claudit makes no other outbound network requests. No analytics, no crash reporting, no CDN pings, no update checks beyond what the Chrome Web Store performs automatically. The **only** outbound destination is the `Base URL` you set in Options.

---

## 6. Data stored locally

Claudit stores the following on your device using Chrome extension APIs. This data **never leaves your device** unless you explicitly send it to the Claude endpoint you configured via a chat.

| Storage | Contents | Lifetime |
|---------|----------|----------|
| `chrome.storage.local` | Credentials (API key or bearer token), `Base URL`, project configuration (specs path, data path), user preferences, test result history, data consent flag | Until you clear it or uninstall |
| `chrome.storage.session` | Agent runtime state (for Service Worker hibernation recovery) | Until browser restart |
| IndexedDB (`claudit-prod-agent`) | File System Access API handles for folders you selected, agent memory entries | Until you clear it or uninstall |
| Local filesystem (via File System Access API) | Test results, regression test YAML files, learning history — written to folders you explicitly selected in Options | Controlled by you |

**File system access:** Claudit can only read and write files in directories you personally selected via the directory picker dialog. You grant this permission explicitly in the Options page, and you can revoke it at any time using the Clear button next to each folder or by clearing the extension's site data.

- **Specs folder** (optional): read-only. Claudit reads `spec.yaml` definition files as test criteria. Nothing is written to this folder.
- **Working folder**: Claudit writes test results, regression YAML files, and agent learning history here. No data from this folder is uploaded externally.

---

## 7. Tracking and analytics

Claudit contains **zero** tracking code. Specifically:

- No Google Analytics, Mixpanel, Amplitude, Sentry, or similar
- No fingerprinting
- No ad networks
- No referrer tracking
- No usage statistics sent anywhere

---

## 8. Permissions we request and why

Claudit requests several Chrome extension permissions. Each is used strictly for the stated purpose.

| Permission | Type | Why |
|-----------|------|-----|
| `declarativeNetRequestWithHostAccess` | **Optional** (granted on demand) | Used to route requests to a custom Base URL/proxy if configured. Not requested unless you set a custom endpoint. |
| `debugger` | Required | Used to drive click/input/screenshot/network capture via Chrome DevTools Protocol, which is more reliable than content-script injection on modern React admin UIs. The yellow "Chromium is being debugged" banner is always shown during active testing. |
| `scripting` | Required | Injects read-only DOM inspection code to capture selectors and form state. |
| `webRequest` | Required | Intercepts XHR/fetch requests from the tab under test to display the Network panel inside the sidebar. Request bodies are not read. |
| `storage` / `alarms` / `tabs` / `sidePanel` / `windows` / `webNavigation` | Required | Standard extension APIs for UI, tab coordination, and Service Worker keepalive. |
| `offscreen` | Required | File System Access API requires a DOM context; the Offscreen Document provides this without opening a visible window. |
| `notifications` | Required | Shown when a long-running test completes. |
| `activeTab` | Required | Allows inspecting the tab currently in focus without requiring broad tab permissions. |

We do not request permissions we do not use.

---

## 9. Your control

You can, at any time:

- **Withdraw data consent** — Options page → About section → uncheck the data processing consent checkbox. The extension will stop sending data to the Claude API.
- **Remove your credentials** — Options page → clear the API key / bearer token fields → save.
- **Change the API endpoint** — Options page → edit the `Base URL` field → save.
- **Revoke folder permissions** — Options page → Folder access section → click Clear next to any folder.
- **Clear all data** — Chrome → `chrome://extensions` → Claudit → Details → "Remove extension" (wipes all storage), or Site Data → "Clear data".
- **Inspect traffic** — Chrome DevTools → Network tab, filter by the host portion of your configured `Base URL`, to see exactly what Claudit sends.

---

## 10. Children's privacy

Claudit is a developer tool not intended for children under 13. We do not knowingly collect any data from anyone.

---

## 11. Changes to this policy

If this policy changes, the "Last updated" date at the top will change, and a notice will be added to the extension's release notes on the Chrome Web Store listing. Material changes will be highlighted. Continued use of Claudit after an update constitutes acceptance of the revised policy.

---

## 12. Contact

Claudit publishes its Privacy Policy, permission justification, and user-facing documentation in a public repository. Questions, concerns, and data requests are handled via that repository's issue tracker:

- **Documentation repository:** https://github.com/youzooyou/claudit
- **Issues:** https://github.com/youzooyou/claudit/issues

If you need a private channel (for example, to report a security issue), open a minimal public issue requesting a private contact method, and we will respond with a secure channel.

---

## 한국어 전문 (Korean full version)

**최종 수정일:** 2026-04-16
**적용 대상:** Claudit Chrome 익스텐션 (버전 1.0.0 이상)

### 1. 개요

Claudit은 로컬 브라우저 자동화 테스트 도구입니다. 모든 기능이 브라우저 내부에서 동작하며, 사용자가 채팅·테스트 세션을 명시적으로 시작했을 때만 **Options 페이지에서 설정한 Claude API 엔드포인트** 한 곳과 통신합니다. 자체 서버를 운영하지 않고, 분석/트래킹/광고를 일절 수행하지 않습니다.

**핵심 요약:**
- 🟢 Claude 엔드포인트로 전송하는 프롬프트를 제외한 모든 데이터는 사용자 기기에만 보관
- 🟢 엔드포인트는 사용자가 선택: Anthropic 공식 API, 사내 프록시, 자체 게이트웨이 모두 가능
- 🟢 데이터 전송 전, Options 페이지에 표시되는 데이터 처리 고지에 명시적으로 동의해야 함
- 🟢 분석/텔레메트리/크래시 리포트 없음
- 🟢 광고/제3자 SDK 없음
- 🟢 자격 증명과 파일 권한은 언제든 사용자가 직접 통제

### 2. Claudit의 용도 (의도된 사용)

**Claudit은 웹 애플리케이션의 End-to-End(E2E) 품질 테스트를 위한 QA 도구입니다.** 소프트웨어 테스터·QA 엔지니어·개발자가 본인이 운영하거나 테스트 권한이 있는 웹 어드민 패널·내부 도구의 동작을 기획서(스펙) 대비 검증하도록 설계되었습니다.

**허용 범위:**
- ✅ 본인·소속 조직이 운영하거나 명시적 테스트 권한을 받은 웹 애플리케이션 테스트
- ✅ 회귀 테스트, 엣지 케이스 탐색, 스펙 준수 검증
- ✅ 버그 리포트용 재현 가능한 테스트 증거(스크린샷·네트워크 로그) 수집
- ✅ 사내 QA 프로세스 일부로 테스트 계획 실행

**허용되지 않는 사용:**
- ❌ 본인 소유가 아닌 제3자 사이트 콘텐츠 스크래핑
- ❌ 타 사이트의 인증·요금제·약관 우회
- ❌ 사용자가 만들지 않은 서비스에 대한 개인 계정 자동화
- ❌ 감시·자격증명 탈취·무단 데이터 추출

모든 자동화는 사용자 브라우저 내부에서 실행되며 원격 오케스트레이션 서버는 존재하지 않습니다. Chrome의 노란색 "debugger attached" 배너가 자동화 활성 상태를 사용자에게 항상 가시화합니다.

### 3. 수집 데이터

**없음.** Claudit은 자체 서버를 운영하지 않으므로 수집·전송·서버 저장하는 데이터가 존재하지 않습니다.

### 4. 사용자 동의

Claudit이 Claude API 엔드포인트로 데이터를 전송하기 전, Options 페이지에 표시되는 데이터 처리 고지에 명시적으로 동의해야 합니다. 고지 내용에는 테스트 세션에서 전송될 수 있는 데이터 유형이 구체적으로 나열되며, 동의 체크박스를 선택해야 핵심 기능이 활성화됩니다.

동의는 Options 페이지에서 체크박스 해제로 언제든 철회할 수 있습니다. 동의 철회 시 로컬에 저장된 데이터는 삭제되지 않습니다.

### 5. 사용자가 설정한 Claude API 엔드포인트로 전송되는 데이터

**전송 대상:** Options 페이지에서 설정한 `Base URL` 한 곳으로만 전송됩니다.

**요청별 전송 데이터:**

| 데이터 | 시점 | 목적 |
|--------|------|------|
| 채팅 메시지 | 매 턴 | 에이전트 프롬프트 |
| 현재 탭의 스크린샷 | `browser_capture` 도구 호출 시 | 시각적 컨텍스트 |
| DOM 스냅샷 (HTML 구조·폼 상태·그리드 내용) | `inspect_page`, `get_form_state` 도구 호출 시 | 페이지 이해 — 비밀번호 필드 제외 |
| 테스트 탭의 네트워크 요청 URL | `browser_network_get` 도구 호출 시 | API 검증 — 요청 바디 및 다른 탭 트래픽 미포함 |
| 로드한 스펙 파일 | 세션 시작 | 테스트 계획 수립 |
| 현재 탭 URL | 매 턴 | 컨텍스트 식별 |

> **네트워크 캡처 참고:** 현재 테스트 중인 탭에서 발생하는 XHR/fetch(API) 요청만 기록됩니다. 이미지·폰트·스크립트·스타일시트 등 정적 리소스와 다른 탭의 요청은 제외됩니다. 요청 바디(body)는 캡처하지 않습니다. 익스텐션 자체의 Claude API 트래픽도 제외됩니다.

**인증 방식:** 다음 중 하나의 자격 증명을 로컬 저장소에서 읽어 요청에 첨부합니다.
- **`x-api-key` 헤더** — Anthropic API 키 (표준)
- **`Authorization: Bearer <token>` 헤더** — 프록시·게이트웨이용 베어러 토큰

이 자격 증명은 사용자가 설정한 `Base URL` 외 어떤 곳으로도 전송되지 않습니다.

**기타 제3자 전송:** 없음. Chrome 웹 스토어 자동 업데이트 체크 외에 어떤 외부 통신도 없습니다.

### 6. 로컬 저장 데이터

| 저장소 | 내용 | 보존 기간 |
|--------|------|----------|
| `chrome.storage.local` | 자격 증명, 프로젝트 설정, 사용자 환경설정, 테스트 결과 이력, 동의 플래그 | 사용자 삭제 또는 익스텐션 제거 시까지 |
| `chrome.storage.session` | 에이전트 런타임 상태 (Service Worker 하이버네이션 복구용) | 브라우저 재시작 시까지 |
| IndexedDB (`claudit-prod-agent`) | 사용자가 선택한 폴더의 File System Access API 핸들, 에이전트 메모리 | 사용자 삭제 또는 익스텐션 제거 시까지 |
| 로컬 파일시스템 (File System Access API) | 테스트 결과, 회귀 테스트 YAML, 학습 이력 | 사용자 통제 |

**파일시스템 접근:**
- **기획서 스펙 폴더** (선택 사항): 읽기 전용. `spec.yaml` 파일을 테스트 기준으로 읽습니다. 이 폴더에는 아무것도 쓰지 않습니다.
- **AI 작업 폴더**: 테스트 결과, 회귀 테스트 YAML, 에이전트 학습 이력을 여기에 씁니다. 이 폴더의 데이터는 외부로 전송되지 않습니다.

### 7. 추적 및 분석

Claudit에는 **어떠한** 트래킹 코드도 없습니다 (Google Analytics / Mixpanel / Amplitude / Sentry 등 없음, 핑거프린팅 없음, 광고 네트워크 없음, 사용 통계 외부 전송 없음).

### 8. 요청하는 권한과 이유

| 권한 | 유형 | 이유 |
|------|------|------|
| `declarativeNetRequestWithHostAccess` | **선택적** (필요 시 요청) | 커스텀 Base URL/프록시 라우팅 전용. 기본 엔드포인트 사용 시 불필요 |
| `debugger` | 필수 | Chrome DevTools Protocol로 클릭·입력·스크린샷 수행. 자동화 중 항상 노란색 배너 표시 |
| `scripting` | 필수 | 읽기 전용 DOM 검사 코드 주입으로 셀렉터·폼 상태 캡처 |
| `webRequest` | 필수 | 테스트 탭의 XHR/fetch 요청을 사이드바 네트워크 패널에 표시. 요청 바디 미읽음 |
| `storage` / `alarms` / `tabs` / `sidePanel` / `windows` / `webNavigation` | 필수 | UI, 탭 조율, Service Worker 유지 등 표준 익스텐션 API |
| `offscreen` | 필수 | File System Access API가 DOM 컨텍스트를 요구하여 Offscreen Document 필요 |
| `notifications` | 필수 | 장시간 실행 테스트 완료 시 사용자 알림 |
| `activeTab` | 필수 | 현재 포커스된 탭 검사 (광범위한 탭 권한 불필요) |

사용하지 않는 권한은 요청하지 않습니다.

### 9. 사용자 통제

사용자는 언제든 다음을 수행할 수 있습니다.

- **데이터 전송 동의 철회** — Options 페이지 → About 섹션 → 동의 체크박스 해제
- **자격 증명 제거** — Options 페이지 → API 키/베어러 토큰 필드 비우고 저장
- **API 엔드포인트 변경** — Options 페이지 → `Base URL` 필드 수정 후 저장
- **폴더 권한 해제** — Options 페이지 → 폴더 접근 권한 섹션 → 각 폴더의 초기화 버튼 클릭
- **모든 데이터 삭제** — Chrome → `chrome://extensions` → Claudit → "익스텐션 제거"
- **트래픽 검사** — Chrome DevTools → Network 탭에서 설정한 `Base URL` 호스트로 필터링

### 10. 아동 개인정보

Claudit은 13세 미만 아동을 대상으로 하지 않는 개발자 도구입니다. 어떤 사용자로부터도 의도적으로 데이터를 수집하지 않습니다.

### 11. 정책 변경

이 정책이 변경될 경우 최상단의 "최종 수정일"이 변경되며, 변경 사항은 Chrome 웹 스토어의 릴리스 노트에 공지됩니다. 정책 변경 후 Claudit 계속 사용은 개정된 정책에 대한 동의로 간주됩니다.

### 12. 연락처

- **문서 저장소:** https://github.com/youzooyou/claudit
- **이슈:** https://github.com/youzooyou/claudit/issues
