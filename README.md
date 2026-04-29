# @orbitconnect/react

React SDK for OrbitConnect — drop in real-time chat, voice/video calls, and meetings into any React app.

> Get your API keys at [orbitconnect.cloud](https://orbitconnect.cloud). All requests go to `https://api.orbitconnect.cloud/api/v1` — this base URL is fixed and cannot be overridden.

## Install

```bash
npm install @orbitconnect/react
```

Also import the stylesheet once in your entry file:

```ts
import '@orbitconnect/react/index.css'
```

---

## Get started

### 1. Create an account

Sign up at [orbitconnect.cloud](https://orbitconnect.cloud) to get your:
- `ORBIT_SECRET_KEY` — server-side secret key (`sk_live_…`)
- `ORBIT_CLIENT_ID` — publishable client ID (`pk_live_…`)

### 2. Backend — issue a user token

```ts
// Express / Next.js API route
import { OrbitServer } from '@orbitconnect/server'

const orbit = new OrbitServer({ secretKey: process.env.ORBIT_SECRET_KEY! })

app.post('/auth/token', async (req, res) => {
  const user = await orbit.users.create({
    external_id:  req.user.id,
    display_name: req.user.name,
  })
  const { token } = await orbit.users.createToken(user.id)
  res.json({ token, userId: user.id, clientId: process.env.ORBIT_CLIENT_ID })
})
```

### 3. Frontend — wrap with OrbitProvider

```tsx
import { OrbitProvider, Chat } from '@orbitconnect/react'
import '@orbitconnect/react/index.css'

export function App() {
  const [auth, setAuth] = useState(null)

  async function login() {
    const data = await fetch('/auth/token', { method: 'POST' }).then(r => r.json())
    setAuth(data)
  }

  if (!auth) return <button onClick={login}>Sign in</button>

  return (
    <OrbitProvider
      clientId={auth.clientId}
      userToken={auth.token}
      appUserId={auth.userId}
    >
      <Chat />
    </OrbitProvider>
  )
}
```

That's it — you now have a fully functional chat with real-time messaging, presence, reactions, file attachments, and voice/video calls.

---

## OrbitProvider

The root context provider. Wrap your app (or the section that uses OrbitConnect) with it once.

```tsx
<OrbitProvider
  clientId="pk_live_..."
  userToken={token}
  appUserId={userId}
  baseTheme={darkTheme}
  theme={{ primary: '#7c3aed', bubbleSender: '#7c3aed' }}
  appConfig={{ name: 'MyApp', showPoweredBy: false }}
  onTokenExpired={async () => fetchFreshToken()}
>
  <App />
</OrbitProvider>
```

| Prop | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | `string` | ✓ | Publishable key (`pk_live_…` or `pk_test_…`) |
| `userToken` | `string` | ✓ | Short-lived token from `orbit.users.createToken()` |
| `appUserId` | `string` | ✓ | The authenticated app user ID |
| `theme` | `PartialTheme` | | CSS variable overrides — see [Theming](#theming) |
| `baseTheme` | `OrbitTheme` | | Base theme object (default: `darkTheme`) |
| `appConfig` | `AppConfig` | | Branding, icons, sounds — see [AppConfig](#appconfig) |
| `onTokenExpired` | `() => Promise<string>` | | Called on WS close 4001 — return a fresh token to reconnect |

---

## Widgets

Drop-in components that work out of the box inside `OrbitProvider`.

### Chat

Full-featured chat UI — conversation list, message bubbles, composer, reactions, file attachments, voice messages, call buttons, and more.

```tsx
import { Chat } from '@orbitconnect/react'

<Chat
  onCallRequest={(userId, type) => initiateCall(userId, type)}
  chatBanner={<div>Support hours: Mon–Fri 9am–5pm</div>}
  defaultQuickMessages={[
    { id: '1', label: 'Greeting', text: 'Hi! How can I help?' },
  ]}
/>
```

| Prop | Type | Description |
| --- | --- | --- |
| `conversationId` | `string` | Open directly to a specific conversation |
| `onCallRequest` | `(userId, type) => void` | Called when the user taps the call/video button |
| `renderHeader` | `(info: ChatHeaderInfo) => ReactNode` | Replace the default header |
| `chatBanner` | `ReactNode` | Rendered above the message list |
| `defaultQuickMessages` | `QuickMessage[]` | Seed the quick-message picker |

Included out of the box: real-time messaging, typing indicators, read receipts, reactions, reply/quote, file/image/video/audio attachments, voice recording, sticker & GIF picker, message search, pin messages, batch select/delete/forward, presence, call history.

### VideoCall

Floating WebRTC call overlay — renders automatically when a call is active.

```tsx
import { VideoCall, useCall } from '@orbitconnect/react'

function App() {
  const { initiateCall } = useCall()
  return (
    <>
      <button onClick={() => initiateCall(userId, 'video')}>Start call</button>
      <VideoCall />  {/* place once at the root */}
    </>
  )
}
```

### MeetingWidget

Full video-conferencing room with participant grid, screen share, reactions, and hand-raise.

```tsx
import { MeetingWidget } from '@orbitconnect/react'

<MeetingWidget
  meetingId={activeMeetingId}
  title="Team Standup"
  onEnd={() => setScreen('lobby')}
/>
```

| Prop | Type | Required | Description |
| --- | --- | --- | --- |
| `meetingId` | `string` | ✓ | The meeting ID to join |
| `title` | `string` | | Display title (default: `"Meeting"`) |
| `onEnd` | `() => void` | | Called when the meeting ends or the user leaves |

---

## Hooks

All hooks must be used inside `OrbitProvider`.

### useMessages(conversationId)

```ts
const {
  messages,           // MessageBubbleProps[]
  loading,
  typingUsers,        // string[] — display names of typing users
  pinnedMessages,
  sendMessage,        // (text, mediaId?, meta?) => Promise<void>
  sendTyping,
  markRead,
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
  localStream,    // MediaStream | null
  remoteStream,   // MediaStream | null
  initiateCall,   // (userId, type, conversationId?) => void
  acceptCall,
  rejectCall,
  endCall,
  toggleMute,     // (muted: boolean) => void
  toggleCamera,   // (enabled: boolean) => void
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
const status = usePresence(userId)  // 'online' | 'away' | 'offline'
```

### useMedia()

```ts
const {
  upload,   // (file: File) => Promise<MediaObject>
  getUrl,   // (mediaId: string) => Promise<string>
} = useMedia()
```

### useRealtime()

```ts
const {
  session,
  createSession,  // (input: CreateSessionInput) => Promise<void>
  endSession,
  transition,     // (type: TransitionType, contextId?: string) => Promise<void>
  emitEvent,      // (eventType: string, payload: object) => Promise<void>
} = useRealtime()
```

### useCallHistory()

```ts
const { records } = useCallHistory()  // CallRecord[]
```

### useOrbitEvent(eventType, handler)

Subscribe to any raw WebSocket event.

```ts
import { useOrbitEvent } from '@orbitconnect/react'

useOrbitEvent('message:new', ({ conversation_id, message }) => {
  console.log('New message:', message)
})

useOrbitEvent('notification:received', ({ payload }) => {
  toast(payload.title as string)
})
```

### useOrbit()

Access the raw context from any child component.

```ts
const { apiClient, appUserId, clientId, baseUrl, appConfig } = useOrbit()
```

---

## Theming

Every colour, radius, and font is a CSS variable you can override.

```tsx
import { OrbitProvider, darkTheme, lightTheme } from '@orbitconnect/react'
import type { PartialTheme } from '@orbitconnect/react'

const myTheme: PartialTheme = {
  primary:                '#7c3aed',
  bubbleSender:           '#7c3aed',
  bubbleSenderForeground: '#fff',
}

<OrbitProvider baseTheme={darkTheme} theme={myTheme} ...>
```

Key token groups: `background`, `foreground`, `card`, `border`, `primary`, `bubbleSender`, `bubbleReceiver`, `composerBg`, `radiusSm/Md/Lg/Xl`, `fontBody`.

---

## AppConfig

White-label every aspect of the UI — name, logo, icons, sounds, and feature flags.

```tsx
<OrbitProvider
  appConfig={{
    name:          'MyApp',
    logo:          <MyLogo />,
    showPoweredBy: false,

    // feature flags
    showAudioIcon:          true,
    showVideoIcon:          true,
    showSearchIcon:         true,
    showAttachMentButton:   true,
    showQuickMessageButton: true,
    useRichTextEditor:      true,
    enableMeetingUi:        true,

    // icon overrides (any ReactNode)
    sendIcon:        <IoPaperPlaneOutline />,
    callIcon:        <IoCallOutline />,
    VideoCallIcon:   <IoVideocamOutline />,

    // custom sounds
    sounds: {
      messageSent:     '/sounds/sent.mp3',
      messageReceived: '/sounds/received.mp3',
      incomingCall:    '/sounds/ring.mp3',
    },
  }}
>
```

---

## Token refresh

When the server closes the WebSocket with code `4001` (token expired), the SDK calls `onTokenExpired`. Return a fresh token to reconnect automatically.

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

## License

MIT — [orbitconnect.cloud](https://orbitconnect.cloud)
