# Changelog

All notable changes to `@orbitconnect/react` will be documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.1.1] — 2025-04-27

### Fixed
- Corrected API base URL from `orbitxyz.muvle.org` to `api.orbitconnect.cloud`
- Fixed unused variable TypeScript errors in `InCallChatPanel.tsx` blocking the build

### Changed
- Updated repository URL to `github.com/core-dynamics/orbitconnect-sdk`
- README now links to [orbitconnect.cloud](https://orbitconnect.cloud) for account creation

---

## [0.1.0] — 2025-04-27

Initial public release.

### Added

**Core**
- `OrbitProvider` — root context provider with WebSocket lifecycle, token refresh (`onTokenExpired`), and theming
- `useOrbit` — access `apiClient`, `appUserId`, `clientId`, `baseUrl`, `appConfig` from any child
- `useAppConfig` — resolved feature flags and icon overrides from `AppConfig`

**Widgets**
- `Chat` — full-featured chat UI: conversation list, message bubbles, composer, reactions, reply/quote, file/image/video/audio attachments, voice recording, emoji & sticker picker, message search, pin, batch select/delete/forward, presence, call history
- `VideoCall` — floating WebRTC call overlay, renders automatically when a call is active
- `MeetingWidget` — full video-conferencing room with participant grid, screen share, reactions, and hand-raise

**Hooks**
- `useMessages` — real-time messages with typing indicators, pinned messages, read receipts, reactions, recall, edit
- `useConversations` — conversation list with live presence, typing, and delivery status
- `useCall` — full call lifecycle: initiate, accept, reject, end, mute, camera toggle
- `useMeeting` — meeting lifecycle: join, leave, mute, screen share, raise hand, reactions
- `useMedia` — file upload with `upload()` and `getUrl()`
- `useNotifications` — in-app notification feed with unread count, mark read, mark all read, clear
- `usePresence` — real-time presence status per user (`online` / `away` / `offline`)
- `useCallHistory` — call records with caller/callee info
- `useRealtime` — session lifecycle with transitions (chat → call → meeting) and custom event emission
- `useOrbitEvent` — subscribe to any raw WebSocket event
- `useOrbitListener` — access the raw socket and connection status

**Meeting context**
- `MeetingProvider`, `MeetingContext`, `useMeetingContext` — standalone meeting context for custom meeting UIs

**Theme**
- `ThemeProvider`, `useTheme`, `darkTheme`, `lightTheme`
- Full CSS variable token set: backgrounds, foregrounds, bubbles, composer, borders, radii, fonts
- `PartialTheme` type for partial overrides via `OrbitProvider`

**Stylesheet**
- `dist/index.css` — pre-built Tailwind CSS stylesheet, import once via `import '@orbitconnect/react/index.css'`

**Build**
- Vite library build (ESM only) with React Compiler (babel-plugin-react-compiler)
- TypeScript declarations emitted to `dist/` via `tsc -p tsconfig.build.json`
- Tailwind CSS minified and bundled to `dist/index.css`

---

[0.1.0]: https://github.com/core-dynamics/orbitconnect-sdk/releases/tag/react-v0.1.0
