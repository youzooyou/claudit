# Claudit Privacy Policy

**Last updated:** 2026-04-11
**Applies to:** Claudit Chrome Extension (version 2.0.0 and later)

---

## 1. Summary

Claudit is a local browser automation testing tool. It runs entirely inside your browser and communicates **only** with the Claude API endpoint you configure in the Options page when you explicitly start a chat or test session. We do not operate servers, we do not collect analytics, and we do not track you.

**Short version:**
- 🟢 All data stays on your device except prompts you send to the Claude endpoint you configured
- 🟢 You choose the endpoint: Anthropic's public API, a corporate proxy, or a self-hosted gateway
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
5. Results are written to a folder the user selected, for later review or CI integration

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

## 4. Data sent to the Claude API endpoint you configured

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
| Screenshots of the current tab | Tool call | Visual context for the model |
| DOM snapshots (HTML structure, form state, grid contents) | Tool call | Page understanding |
| Network requests captured in the active tab | Tool call | API verification |
| Specification files you load | Session start | Test planning |
| URL of the active tab | Session start | Context identification |

**Authentication:** One of the following credentials is read from local storage and attached to each request:
- **`x-api-key`** header — your Anthropic API key (standard), *or*
- **`Authorization: Bearer <token>`** header — a bearer token, typically used for proxies or gateways that require token-based auth

These credentials are never transmitted anywhere except the `Base URL` you configured. If the default `https://api.anthropic.com` is used, the relevant governing policy is [Anthropic's Privacy Policy](https://www.anthropic.com/legal/privacy). If you use a custom endpoint, the governing policy is whatever agreement you have with that endpoint's operator.

### No other third parties

Claudit makes no other outbound network requests. No analytics, no crash reporting, no CDN pings, no update checks beyond what the Chrome Web Store performs automatically. The **only** outbound destination is the `Base URL` you set in Options.

---

## 5. Data stored locally

Claudit stores the following on your device using Chrome extension APIs. This data **never leaves your device** unless you explicitly send it to the Claude endpoint you configured via a chat.

| Storage | Contents | Lifetime |
|---------|----------|----------|
| `chrome.storage.local` | Credentials (API key or bearer token), `Base URL`, project configuration (specs path, data path), user preferences, test result history | Until you clear it or uninstall |
| `chrome.storage.session` | Agent runtime state (for Service Worker hibernation recovery) | Until browser restart |
| IndexedDB (`claudit-prod-agent`) | File System Access API handles for folders you selected, agent memory entries | Until you clear it or uninstall |
| Local filesystem (via File System Access API) | Test results, regression test YAML files, learning history — written to folders you explicitly selected in Options | Controlled by you |

**File system access:** Claudit can only read and write files in directories you personally selected via the directory picker dialog. You grant this permission explicitly in the Options page, and you can revoke it at any time by clearing the extension's site data.

---

## 6. Tracking and analytics

Claudit contains **zero** tracking code. Specifically:

- No Google Analytics, Mixpanel, Amplitude, Sentry, or similar
- No fingerprinting
- No ad networks
- No referrer tracking
- No usage statistics sent anywhere

---

## 7. Permissions we request and why

Claudit requests several Chrome extension permissions. Each is used strictly for the stated purpose. Full justification is available at [`docs/permissions.md`](./docs/permissions.md).

Summary:
- **`<all_urls>` host permission** — required because users test arbitrary web admin panels; we cannot enumerate target domains in advance
- **`debugger`** — used to drive click/input/screenshot via Chrome DevTools Protocol, which is more reliable than content-script injection on modern React admins
- **`scripting`** — used to inject read-only DOM inspection code to capture selectors and form state
- **`storage` / `alarms` / `tabs` / `webRequest` / `sidePanel` / `windows`** — standard extension APIs for UI, networking capture, and tab coordination
- **`offscreen`** — required because the File System Access API needs a DOM context
- **`notifications`** — shown when a long-running test completes

We do not request permissions we do not use.

---

## 8. Your control

You can, at any time:

- **Remove your credentials** — Options page → clear the API key / bearer token fields → save. All future requests will fail until new credentials are provided.
- **Change the API endpoint** — Options page → edit the `Base URL` field → save. All future requests will go to the new URL.
- **Revoke folder permissions** — Options page → reselect or clear directory handles.
- **Clear all data** — Chrome → `chrome://extensions` → Claudit → Details → "Remove extension" (wipes all storage), or Site Data → "Clear data".
- **Inspect traffic** — Chrome DevTools → Network tab, filter by the host portion of your configured `Base URL`, to see exactly what Claudit sends.

---

## 9. Children's privacy

Claudit is a developer tool not intended for children under 13. We do not knowingly collect any data from anyone.

---

## 10. Changes to this policy

If this policy changes, the "Last updated" date at the top will change, and a notice will be added to the extension's release notes on the Chrome Web Store listing. Material changes will be highlighted. Continued use of Claudit after an update constitutes acceptance of the revised policy.

---

## 11. Contact

Claudit publishes its Privacy Policy, permission justification, and user-facing documentation in a public repository. Questions, concerns, and data requests are handled via that repository's issue tracker:

- **Documentation repository:** https://github.com/youzooyou/claudit
- **Issues:** https://github.com/youzooyou/claudit/issues

If you need a private channel (for example, to report a security issue), open a minimal public issue requesting a private contact method, and we will respond with a secure channel.

---

## 한국어 전문 (Korean full version)

**최종 수정일:** 2026-04-11
**적용 대상:** Claudit Chrome 익스텐션 (버전 2.0.0 이상)

### 1. 개요

Claudit은 로컬 브라우저 자동화 테스트 도구입니다. 모든 기능이 브라우저 내부에서 동작하며, 사용자가 채팅·테스트 세션을 명시적으로 시작했을 때만 **Options 페이지에서 설정한 Claude API 엔드포인트** 한 곳과 통신합니다. 자체 서버를 운영하지 않고, 분석/트래킹/광고를 일절 수행하지 않습니다.

**핵심 요약:**
- 🟢 Claude 엔드포인트로 전송하는 프롬프트를 제외한 모든 데이터는 사용자 기기에만 보관
- 🟢 엔드포인트는 사용자가 선택: Anthropic 공식 API, 사내 프록시, 자체 게이트웨이 모두 가능
- 🟢 분석/텔레메트리/크래시 리포트 없음
- 🟢 광고/제3자 SDK 없음
- 🟢 자격 증명과 파일 권한은 언제든 사용자가 직접 통제

### 2. Claudit의 용도 (의도된 사용)

**Claudit은 웹 애플리케이션의 End-to-End(E2E) 품질 테스트를 위한 QA 도구입니다.** 소프트웨어 테스터·QA 엔지니어·개발자가 본인이 운영하거나 테스트 권한이 있는 웹 어드민 패널·내부 도구의 동작을 기획서(스펙) 대비 검증하도록 설계되었습니다.

**대표적 워크플로:**
1. QA 엔지니어가 테스트 권한이 있는 스테이징/운영 어드민 화면을 연다
2. 해당 화면의 기대 동작을 기술한 `spec.yaml` 스펙 파일을 로드한다
3. Claudit에 회귀 테스트 실행 또는 엣지 케이스 탐색을 지시한다
4. Claudit이 화면을 직접 조작(클릭·입력·스크린샷·네트워크 관찰)하며 스펙 대비 동작을 비교하고 합격/실패 결과를 보고한다
5. 결과는 사용자가 선택한 폴더에 기록되어 후속 리뷰·CI 연동에 사용된다

**허용 범위 (의도된 사용):**
- ✅ 본인·소속 조직이 운영하거나 명시적 테스트 권한을 받은 웹 애플리케이션 테스트
- ✅ 회귀 테스트, 엣지 케이스 탐색, 스펙 준수 검증
- ✅ 버그 리포트용 재현 가능한 테스트 증거(스크린샷·네트워크 로그) 수집
- ✅ 사내 QA 프로세스 일부로 테스트 계획 실행

**허용되지 않는 사용 (설계·지원 대상 아님):**
- ❌ 본인 소유가 아닌 제3자 사이트 콘텐츠 스크래핑
- ❌ 타 사이트의 인증·요금제·약관 우회
- ❌ 사용자가 만들지 않은 서비스에 대한 개인 계정 자동화
- ❌ 감시·자격증명 탈취·무단 데이터 추출

모든 자동화는 사용자 브라우저 내부에서 실행되며 원격 오케스트레이션 서버는 존재하지 않습니다. Claudit은 비감독 상태에서 동작할 수 없으며, 모든 테스트 세션은 사용자의 명시적 채팅 메시지로 시작됩니다. Chrome의 노란색 "debugger attached" 배너가 자동화 활성 상태를 사용자에게 항상 가시화합니다.

### 3. 수집 데이터

**없음.** Claudit은 자체 서버를 운영하지 않으므로 수집·전송·서버 저장하는 데이터가 존재하지 않습니다.

### 4. 사용자가 설정한 Claude API 엔드포인트로 전송되는 데이터

**전송 대상:** Options 페이지에서 설정한 `Base URL` 한 곳으로만 전송됩니다. 이 엔드포인트는 사용자가 직접 통제합니다.

- **기본값:** `https://api.anthropic.com` (Anthropic 공식 Claude API)
- **변경 가능:** Base URL 필드에 임의의 Claude 호환 엔드포인트 지정 가능. 예:
  - 조직 로깅·쿼터·컴플라이언스 정책을 적용하는 사내 프록시
  - Anthropic 앞에 두는 자체 호스팅 또는 프라이빗 게이트웨이
  - 별도 데이터 거주성 보장을 갖는 지역/정부용 변종

**Claudit은 전송 대상 서버를 선택하지 않고 그 신원을 검증하지도 않습니다.** 사용자가 `Base URL`을 변경하면 이후 모든 요청이 해당 URL로 전송되며, Claudit은 경고·로그·우회 처리를 하지 않습니다. 선택한 엔드포인트의 프라이버시·로깅·보관 정책을 이해할 책임은 사용자(및 소속 조직)에게 있습니다.

**요청별 전송 데이터:**

| 데이터 | 시점 | 목적 |
|--------|------|------|
| 채팅 메시지 | 매 턴 | 에이전트 프롬프트 |
| 현재 탭의 스크린샷 | 도구 호출 | 시각적 컨텍스트 |
| DOM 스냅샷 (HTML 구조·폼 상태·그리드 내용) | 도구 호출 | 페이지 이해 |
| 현재 탭에서 캡처한 네트워크 요청 | 도구 호출 | API 검증 |
| 로드한 스펙 파일 | 세션 시작 | 테스트 계획 수립 |
| 현재 탭 URL | 세션 시작 | 컨텍스트 식별 |

**인증 방식:** 다음 중 하나의 자격 증명을 로컬 저장소에서 읽어 요청에 첨부합니다.
- **`x-api-key` 헤더** — Anthropic API 키 (표준)
- **`Authorization: Bearer <token>` 헤더** — 프록시·게이트웨이가 요구하는 베어러 토큰

이 자격 증명은 사용자가 설정한 `Base URL` 외 어떤 곳으로도 전송되지 않습니다. 기본 `https://api.anthropic.com`을 사용한다면 [Anthropic 개인정보처리방침](https://www.anthropic.com/legal/privacy)이 적용되고, 커스텀 엔드포인트를 사용한다면 해당 엔드포인트 운영자와의 계약이 적용됩니다.

**기타 제3자 전송:** 없음. 분석/크래시 리포트/CDN 핑/업데이트 체크 모두 없으며, Chrome 웹 스토어가 자동으로 수행하는 업데이트 체크 외에 어떤 외부 통신도 발생하지 않습니다.

### 5. 로컬 저장 데이터

아래 데이터는 Chrome 익스텐션 API로 사용자 기기에 저장됩니다. 사용자가 채팅을 통해 명시적으로 Claude에게 보내지 않는 한 **기기 외부로 유출되지 않습니다.**

| 저장소 | 내용 | 보존 기간 |
|--------|------|----------|
| `chrome.storage.local` | 자격 증명(API 키 또는 베어러 토큰), 프로젝트 설정(스펙·데이터 경로), 사용자 환경설정, 테스트 결과 이력 | 사용자 삭제 또는 익스텐션 제거 시까지 |
| `chrome.storage.session` | 에이전트 런타임 상태 (Service Worker 하이버네이션 복구용) | 브라우저 재시작 시까지 |
| IndexedDB (`claudit-prod-agent`) | 사용자가 선택한 폴더의 File System Access API 핸들, 에이전트 메모리 | 사용자 삭제 또는 익스텐션 제거 시까지 |
| 로컬 파일시스템 (File System Access API) | 테스트 결과, 회귀 테스트 YAML, 학습 이력 — 사용자가 Options에서 명시적으로 선택한 폴더에만 기록 | 사용자 통제 |

**파일시스템 접근:** Claudit은 사용자가 디렉토리 선택 다이얼로그에서 직접 선택한 폴더에만 읽기/쓰기할 수 있습니다. 이 권한은 Options 페이지에서 명시적으로 부여되며, 익스텐션 사이트 데이터를 삭제하면 언제든 회수됩니다.

### 6. 추적 및 분석

Claudit에는 **어떠한** 트래킹 코드도 없습니다.

- Google Analytics / Mixpanel / Amplitude / Sentry 등 없음
- 핑거프린팅 없음
- 광고 네트워크 없음
- 레퍼러 추적 없음
- 사용 통계 외부 전송 없음

### 7. 요청하는 권한과 이유

Claudit은 Chrome 익스텐션 권한 여러 개를 요청하며, 각 권한은 명시된 목적으로만 사용됩니다. 전체 근거는 [`docs/permissions.md`](./docs/permissions.md) 참조.

요약:
- **`<all_urls>` 호스트 권한** — 사용자가 테스트하는 어드민 도메인을 사전 열거할 수 없기 때문
- **`debugger`** — Chrome DevTools Protocol로 클릭·입력·스크린샷을 수행. React 어드민에서 content-script 주입보다 훨씬 신뢰성 높음
- **`scripting`** — 읽기 전용 DOM 검사 코드를 주입해 셀렉터와 폼 상태 캡처
- **`storage` / `alarms` / `tabs` / `webRequest` / `sidePanel` / `windows`** — UI·네트워크 캡처·탭 조율을 위한 표준 익스텐션 API
- **`offscreen`** — File System Access API가 DOM 컨텍스트를 요구하기 때문
- **`notifications`** — 장시간 실행 테스트 완료 시 사용자 알림

사용하지 않는 권한은 요청하지 않습니다.

### 8. 사용자 통제

사용자는 언제든 다음을 수행할 수 있습니다.

- **자격 증명 제거** — Options 페이지 → API 키/베어러 토큰 필드 비우고 저장 → 이후 요청은 새 자격 증명이 제공될 때까지 모두 실패
- **API 엔드포인트 변경** — Options 페이지 → `Base URL` 필드 수정 후 저장 → 이후 모든 요청이 새 URL로 전송
- **폴더 권한 해제** — Options 페이지 → 디렉토리 핸들 재선택 또는 제거
- **모든 데이터 삭제** — Chrome → `chrome://extensions` → Claudit → 상세 → "익스텐션 제거"로 모든 저장소 삭제, 또는 사이트 데이터 → "데이터 삭제"
- **트래픽 검사** — Chrome DevTools → Network 탭에서 설정한 `Base URL` 호스트로 필터링하여 Claudit이 무엇을 보내는지 직접 확인

### 9. 아동 개인정보

Claudit은 13세 미만 아동을 대상으로 하지 않는 개발자 도구입니다. 어떤 사용자로부터도 의도적으로 데이터를 수집하지 않습니다.

### 10. 정책 변경

이 정책이 변경될 경우 최상단의 "최종 수정일"이 변경되며, 변경 사항은 Chrome 웹 스토어의 릴리스 노트에 공지됩니다. 중대한 변경은 별도로 강조됩니다. 정책 변경 후 Claudit 계속 사용은 개정된 정책에 대한 동의로 간주됩니다.

### 11. 연락처

Claudit은 Privacy Policy, 권한 근거, 사용자용 문서를 공개 저장소에 게시합니다. 문의·우려·데이터 요청은 해당 저장소의 이슈 트래커를 통해 처리됩니다.

- **문서 저장소:** https://github.com/youzooyou/claudit
- **이슈:** https://github.com/youzooyou/claudit/issues

비공개 채널이 필요한 경우(예: 보안 취약점 신고), 최소한의 공개 이슈에 "비공개 연락 방법 요청"이라고 남겨주시면 안전한 채널로 회신드립니다.
