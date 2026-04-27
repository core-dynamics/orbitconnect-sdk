# @orbitconnect/react-native

React Native SDK for OrbitConnect — real-time chat, voice/video calls, and meetings.

> The SDK connects exclusively to `https://orbitxyz.muvle.org/api/v1`. This base URL is hardcoded and cannot be overridden by consumer apps.

## Install

```bash
npm install @orbitconnect/react-native

# Native deps (link automatically with Expo or bare RN)
npm install react-native-webrtc react-native-sound react-native-audio-recorder-player \
  react-native-image-picker react-native-document-picker react-native-video \
  react-native-fast-image @react-native-clipboard/clipboard \
  react-native-haptic-feedback react-native-reanimated \
  react-native-gesture-handler @shopify/flash-list dayjs
```

### iOS
```bash
cd ios && pod install
```

Add to `Info.plist`:
```xml
<key>NSCameraUsageDescription</key><string>Video calls</string>
<key>NSMicrophoneUsageDescription</key><string>Voice calls and messages</string>
```

### Android
Add to `AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
```

## Quick start

```tsx
import { OrbitProvider, useMessages, useConversations, useCall } from '@orbitconnect/react-native'

export default function App() {
  return (
    <OrbitProvider
      clientId="pk_live_..."
      userToken={token}
      appUserId={userId}
      appConfig={{ name: 'MyApp', showPoweredBy: false }}
    >
      <ChatScreen />
    </OrbitProvider>
  )
}

function ChatScreen() {
  const { conversations } = useConversations()
  const { messages, sendMessage } = useMessages(conversationId)
  const { call, initiateCall, acceptCall, endCall } = useCall()
  // ...
}
```

## OrbitProvider props

| Prop | Type | Required | Description |
|---|---|---|---|
| `clientId` | `string` | ✓ | Publishable key (`pk_live_…` / `pk_test_…`) |
| `userToken` | `string` | ✓ | Short-lived token from `orbit.users.createToken()` |
| `appUserId` | `string` | ✓ | Authenticated app user ID |
| `theme` | `PartialTheme` | | Theme overrides |
| `baseTheme` | `OrbitTheme` | | Base theme (default: `darkTheme`) |
| `onTokenExpired` | `() => Promise<string>` | | Called on WS close 4001 — return a fresh token |
| `appConfig` | `AppConfig` | | Branding / white-label config |

> `baseUrl` is not a configurable prop. All requests go to `https://orbitxyz.muvle.org/api/v1`.

## WebRTC (calls)

WebRTC is fully native via `react-native-webrtc`. The `CallProvider` (inside `OrbitProvider`) handles the full lifecycle automatically — incoming call detection, offer/answer/ICE exchange over WebSocket, and stream management.

```tsx
const { call, localStream, remoteStream, initiateCall, acceptCall, endCall, toggleMute, switchCamera } = useCall()
```

## Audio recording

```tsx
import { useAudioRecorder } from '@orbitconnect/react-native'

const { isRecording, durationMs, startRecording, stopRecording } = useAudioRecorder()

const result = await stopRecording()
// result.uri — local file path to upload
// result.waveform — amplitude array for waveform display
// result.durationMs
```

## Notifications

```tsx
import { useNotifications } from '@orbitconnect/react-native'

const { notifications, unreadCount, markRead, clearAll } = useNotifications()
```

## Third-party library mapping

| Feature | Library |
|---|---|
| WebRTC (calls + meetings) | `react-native-webrtc` |
| Notification sounds | `react-native-sound` |
| Voice message recording | `react-native-audio-recorder-player` |
| Image/video picker | `react-native-image-picker` |
| File picker | `react-native-document-picker` |
| Video playback | `react-native-video` |
| Image display | `react-native-fast-image` |
| Clipboard | `@react-native-clipboard/clipboard` |
| Haptic feedback | `react-native-haptic-feedback` |
| Animations | `react-native-reanimated` |
| Gestures | `react-native-gesture-handler` |
| Virtualized lists | `@shopify/flash-list` |
| Date formatting | `dayjs` |
