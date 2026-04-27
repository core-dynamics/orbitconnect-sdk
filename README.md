# @orbitconnect/react-native

React Native SDK for OrbitConnect — real-time chat, voice/video calls, and meetings.

> All requests go to `https://api.orbitconnect.cloud/api/v1`. This base URL is fixed and cannot be overridden.

## Install

```bash
npm install @orbitconnect/react-native
```

### Peer dependencies

```bash
npm install react-native-webrtc react-native-sound react-native-audio-recorder-player \
  react-native-image-picker react-native-document-picker react-native-video \
  react-native-fast-image @react-native-clipboard/clipboard \
  react-native-haptic-feedback react-native-reanimated \
  react-native-gesture-handler @shopify/flash-list \
  react-native-callkeep react-native-voip-push-notification \
  @react-native-firebase/messaging @notifee/react-native \
  @react-native-async-storage/async-storage dayjs
```

### iOS

```bash
cd ios && pod install
```

Add to `Info.plist`:

```xml
<key>NSCameraUsageDescription</key><string>Video calls</string>
<key>NSMicrophoneUsageDescription</key><string>Voice calls and messages</string>
<key>NSPhotoLibraryUsageDescription</key><string>Send photos</string>
```

### Android

Add to `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.USE_FULL_SCREEN_INTENT" />
```

---

## Quick start

```tsx
import { OrbitProvider, Chat } from '@orbitconnect/react-native'

export default function App() {
  return (
    <OrbitProvider
      clientId="pk_live_..."
      userToken={token}
      appUserId={userId}
      appConfig={{ name: 'MyApp', showPoweredBy: false }}
    >
      <Chat />
    </OrbitProvider>
  )
}
```

---

## OrbitProvider

Wrap your app (or the section that uses OrbitConnect) once.

```tsx
<OrbitProvider
  clientId="pk_live_..."
  userToken={token}
  appUserId={userId}
  baseTheme={darkTheme}
  theme={{ primary: '#7c3aed' }}
  appConfig={{ name: 'MyApp', showPoweredBy: false }}
  onTokenExpired={async () => fetchFreshToken()}
>
  <App />
</OrbitProvider>
```

| Prop | Type | Required | Description |
|---|---|---|---|
| `clientId` | `string` | ✓ | Publishable key (`pk_live_…` / `pk_test_…`) |
| `userToken` | `string` | ✓ | Short-lived token from `orbit.users.createToken()` |
| `appUserId` | `string` | ✓ | Authenticated app user ID |
| `theme` | `PartialTheme` | | Theme overrides |
| `baseTheme` | `OrbitTheme` | | Base theme (default: `darkTheme`) |
| `onTokenExpired` | `() => Promise<string>` | | Called on WS close 4001 — return a fresh token |
| `appConfig` | `AppConfig` | | Branding / white-label config |

---

## Widgets

Drop-in components that work out of the box inside `OrbitProvider`.

### Chat

Full-featured chat UI — conversation list, message bubbles, composer, reactions, file attachments, voice messages, call buttons, and more.

```tsx
import { Chat } from '@orbitconnect/react-native'

<Chat conversationId={id} />
```

| Prop | Type | Description |
|---|---|---|
| `conversationId` | `string` | Open directly to a specific conversation |
| `onCallRequest` | `(userId, type) => void` | Called when the user taps the call/video button |
| `renderHeader` | `(info: ChatHeaderInfo) => ReactNode` | Replace the default header |

### VideoCall

Floating WebRTC call overlay — renders automatically when a call is active.

```tsx
import { VideoCall } from '@orbitconnect/react-native'

// Place once at the root, inside OrbitProvider
<VideoCall />
```

### MeetingWidget

Full video-conferencing room with participant grid, reactions, and hand-raise.

```tsx
import { MeetingWidget } from '@orbitconnect/react-native'

<MeetingWidget
  meetingId={activeMeetingId}
  title="Team Standup"
  onEnd={() => setScreen('lobby')}
/>
```

| Prop | Type | Required | Description |
|---|---|---|---|
| `meetingId` | `string` | ✓ | The meeting ID to join |
| `title` | `string` | | Display title |
| `onEnd` | `() => void` | | Called when the meeting ends or the user leaves |

---

## Hooks

All hooks must be used inside `OrbitProvider`.

### useMessages(conversationId)

```ts
const {
  messages,        // NativeMessage[]
  loading,
  typingUsers,     // string[] — display names of typing users
  pinnedMessages,  // NativeMessage[]
  sendMessage,     // (text, mediaId?, meta?) => Promise<void>
  sendTyping,
  markRead,        // (messageId: string) => void
  editMessageLocally,
} = useMessages(conversationId)
```

### useConversations()

```ts
const {
  conversations,  // Conversation[]
  loading,
  createDirect,   // (participantId: string) => Promise<string>
  refresh,
} = useConversations()
```

### useCall()

```ts
const {
  call,           // ActiveCall | null
  localStream,
  remoteStream,
  initiateCall,   // (userId, type, conversationId?) => void
  acceptCall,
  rejectCall,
  endCall,
  toggleMute,     // (muted: boolean) => void
  toggleCamera,   // (enabled: boolean) => void
  switchCamera,
  dismissEnded,
} = useCall()
```

### useMeeting()

```ts
const {
  meeting,          // ActiveMeeting | null
  joinMeeting,      // (meetingId, title) => Promise<void>
  leaveMeeting,
  muteSelf,
  unmuteSelf,
  startScreenShare,
  stopScreenShare,
  raiseHand,
  lowerHand,
  sendReaction,     // (emoji: string) => void
} = useMeeting()
```

### useAudioRecorder()

```ts
const {
  isRecording,
  durationMs,
  startRecording,
  stopRecording,    // () => Promise<RecordingResult>
  cancelRecording,
} = useAudioRecorder()

// RecordingResult
// { uri: string, durationMs: number, waveform: number[] }
```

### useMedia()

```ts
const {
  upload,           // (localUri, mimeType, filename) => Promise<MediaObject>
  uploadRecording,  // (localUri, mimeType?, filename?) => Promise<MediaObject>
  uploadAndWait,    // upload + wait for server processing
  getMedia,         // (mediaId: string) => Promise<MediaObject>
  getDownloadUrl,   // (mediaId: string) => Promise<string>
  isUploading,
  isProcessing,
  error,
} = useMedia()
```

### useNotifications()

```ts
const {
  notifications,  // OrbitNotification[]
  unreadCount,
  markRead,       // (id: string) => void
  markAllRead,
  clearAll,
} = useNotifications()
```

### usePresence(userId)

```ts
const { status } = usePresence(userId)
// status: 'online' | 'away' | 'offline'
```

### useCallHistory()

```ts
const { records, loading } = useCallHistory()
// records: CallRecord[]
```

### useRealtime()

```ts
const {
  session,
  createSession,  // (input: CreateSessionInput) => Promise<void>
  endSession,
  transition,     // (type: TransitionType, contextId?) => Promise<void>
  isLoading,
  error,
} = useRealtime()
```

### useWebRTC(options)

Low-level hook for custom call UIs.

```ts
const { startCall, answerCall, toggleMute, toggleCamera, switchCamera, cleanup } = useWebRTC({
  callId,
  onRemoteStream: (stream) => setRemoteStream(stream),
  onConnected: () => console.log('connected'),
})
```

### useMeetingWebRTC(meetingId, appUserId)

Low-level full-mesh WebRTC for custom meeting UIs.

```ts
const {
  startLocalStream,
  initiateOffer,
  remoteStreams,   // RemoteStream[] — { userId, stream }[]
  localStream,
  toggleMute,
  toggleCamera,
  switchCamera,
  cleanup,
} = useMeetingWebRTC(meetingId, appUserId)
```

### useRecording(sessionId, sessionType)

Server-side call/meeting recording (not device audio — use `useAudioRecorder` for that).

```ts
const {
  isRecording,
  recordings,      // RecordingInfo[]
  startRecording,
  stopRecording,
  fetchRecordings,
  error,
} = useRecording(sessionId, 'call' | 'meeting')
```

### useSound()

```ts
const { play, stopRingtone } = useSound()

play('incomingCall')   // plays ringtone + vibration
play('messageReceived')
stopRingtone()
```

### useAppBrand()

Returns the resolved brand node based on `appConfig` flags (`showNameOnly`, `showLogoOnly`, etc.).

```ts
const brand = useAppBrand()  // ReactNode
```

### useOrbitEvent(eventType, handler)

Subscribe to any raw WebSocket event.

```ts
import { useOrbitEvent } from '@orbitconnect/react-native'

useOrbitEvent('message:new', ({ conversation_id, message }) => {
  console.log('New message:', message)
})
```

### useOrbit()

Access the raw context from any child component.

```ts
const { apiClient, appUserId, clientId, baseUrl, appConfig } = useOrbit()
```

---

## Background & push notifications

### Setup (Android)

```ts
import { initCallKeep, registerBackgroundCallNotificationHandler } from '@orbitconnect/react-native'

// In your app entry point (index.js)
registerBackgroundCallNotificationHandler()

// In your root component
await initCallKeep({ appName: 'MyApp' })
```

### Setup (iOS)

CallKit is handled automatically via `react-native-callkeep`. Ensure `VoIP` background mode is enabled in Xcode capabilities.

### Incoming call push (manual handling)

```ts
import { handleIncomingPush, showIncomingCallNotification } from '@orbitconnect/react-native'

// From your FCM/APNs handler
await showIncomingCallNotification({
  call_id:     data.call_id,
  caller_id:   data.caller_id,
  caller_name: data.caller_name,
  type:        'voice',
})
```

### Call notification theme (Android)

```tsx
<OrbitProvider
  appConfig={{
    callNotification: {
      backgroundColor: '#1a1a2e',
      acceptColor:     '#4caf50',
      declineColor:    '#f44336',
      acceptLabel:     'Accept',
      declineLabel:    'Decline',
    },
  }}
>
```

### Background service (deprecated)

`backgroundService` / `useBackgroundServiceStatus` / `registerBackgroundCallHandler` are kept for backwards compatibility. Prefer CallKeep (`initCallKeep`) for new integrations.

---

## Theming

```tsx
import { OrbitProvider, darkTheme, lightTheme } from '@orbitconnect/react-native'
import type { PartialTheme } from '@orbitconnect/react-native'

const myTheme: PartialTheme = {
  primary:      '#7c3aed',
  bubbleSender: '#7c3aed',
}

<OrbitProvider baseTheme={darkTheme} theme={myTheme} ...>
```

---

## AppConfig

White-label the UI — name, logo, icons, sounds, and feature flags.

```tsx
<OrbitProvider
  appConfig={{
    name:          'MyApp',
    logo:          <MyLogo />,
    showPoweredBy: false,

    // Feature flags
    showChatHeader:         true,
    showAudioIcon:          true,
    showVideoIcon:          true,
    showSearchIcon:         true,
    showAttachMentButton:   true,
    showQuickMessageButton: true,
    useRichTextEditor:      true,
    enableMeetingUi:        true,

    // Icon overrides (any ReactNode)
    sendIcon:     <IoSend />,
    callIcon:     <IoCall />,
    VideoCallIcon: <IoVideocam />,

    // Custom sounds (local file paths or remote URLs)
    sounds: {
      messageSent:     require('./sounds/sent.mp3'),
      messageReceived: require('./sounds/received.mp3'),
      incomingCall:    require('./sounds/ring.mp3'),
    },

    // In-app notification toasts
    notifications: {
      position:    'top',
      duration:    4000,
      maxVisible:  3,
    },
  }}
>
```

---

## Token refresh

When the server closes the WebSocket with code `4001`, the SDK calls `onTokenExpired`. Return a fresh token to reconnect automatically.

```tsx
<OrbitProvider
  onTokenExpired={async () => {
    const res  = await fetch('/auth/refresh', { method: 'POST' })
    const data = await res.json()
    return data.token
  }}
  ...
>
```

---

## Direct API access

```ts
import { ApiClient } from '@orbitconnect/react-native'

const client = new ApiClient({ baseUrl, userToken, clientId, appUserId })
const data = await client.get('/conversations')
```

---

## Third-party library mapping

| Feature | Library |
|---|---|
| WebRTC (calls + meetings) | `react-native-webrtc` |
| Notification sounds | `react-native-sound` |
| System sounds / ringtone | `@dashdoc/react-native-system-sounds` |
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
| CallKit / ConnectionService | `react-native-callkeep` |
| VoIP push (iOS) | `react-native-voip-push-notification` |
| FCM push (Android) | `@react-native-firebase/messaging` |
| In-app notifications | `@notifee/react-native` |
| Async storage | `@react-native-async-storage/async-storage` |
| Date formatting | `dayjs` |

---

## License

MIT
