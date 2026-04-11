# Task 03 — Chat Panel UI

## Goal
Build the AI chat panel that lets users converse with the AI to tailor their resume. When the AI responds with suggested edits, an **Apply Changes** button appears so the user can write the edits directly to the PDF.

## Dependencies
- Task 01 complete (Vite project running)
- Task 02 complete (WebViewer instance in Zustand store)
- Task 04 complete (backend `/api/chat` endpoint available), or mock the API call during development

## Files to Create / Modify
```
client/src/
├── components/
│   └── ChatPanel.jsx        # Chat UI component
├── services/
│   └── aiService.js         # Axios call to backend /api/chat
└── store/
    └── chatStore.js         # Zustand store slice for chat state
```

## Steps

### 1. Zustand Store — `store/chatStore.js`
Holds:
- `messages` — array of `{ role: 'user' | 'assistant', content: string, edits?: EditItem[] }`
- `isLoading` — boolean for pending API call
- `addMessage(message)` — append to history
- `setLoading(bool)` — toggle loading state

```js
// store/chatStore.js
import { create } from 'zustand';

const useChatStore = create((set) => ({
  messages: [],
  isLoading: false,
  addMessage: (message) =>
    set((state) => ({ messages: [...state.messages, message] })),
  setLoading: (isLoading) => set({ isLoading }),
}));

export default useChatStore;
```

### 2. AI Service — `services/aiService.js`
Wraps the backend API call:
- `sendMessage({ messages, resumeText })` → `Promise<{ reply: string, edits?: EditItem[] }>`
- Uses `axios` with `VITE_API_BASE_URL` from env.

```js
// services/aiService.js
import axios from 'axios';

const api = axios.create({ baseURL: import.meta.env.VITE_API_BASE_URL });

export async function sendMessage({ messages, resumeText }) {
  const res = await api.post('/api/chat', { messages, resumeText });
  return res.data; // { reply, edits }
}
```

### 3. ChatPanel Component — `components/ChatPanel.jsx`
Layout:
```
┌─────────────────────────┐
│  Message history (scroll)│
│  ...                    │
│  [AI reply]             │
│  ┌─────────────────┐    │
│  │ Apply Changes   │    │  ← shown only when edits present
│  └─────────────────┘    │
├─────────────────────────┤
│  [Text input]  [Send]   │
└─────────────────────────┘
```

Behavior:
1. User types a message and clicks **Send** (or presses Enter).
2. Component reads the current resume text from the PDF via WebViewer:
   ```js
   const instance = useViewerStore.getState().instance;
   const { documentViewer } = instance.Core;
   const text = await documentViewer.getDocument().loadPageText(1); // or all pages
   ```
3. Calls `aiService.sendMessage({ messages, resumeText })`.
4. Appends the AI reply (and optional `edits`) to the `messages` list in the Zustand store.
5. If `edits` is present, renders the **Apply Changes** button beneath that message.
6. On **Apply Changes** click:
   - Calls `applyEdits(edits)` from the `useWebViewer` hook (exposed via Zustand store or prop).
   - Marks the message as applied (disable the button, show a checkmark).

### 4. Wire into App Layout
Update `App.jsx` to render `ChatPanel` in the right panel.

## EditItem Type
```ts
interface EditItem {
  find: string;    // exact text to find in the PDF
  replace: string; // replacement text
}
```

## Acceptance Criteria
- User can send messages and see a scrolling conversation history.
- The AI reply appears after the API call completes; a loading spinner shows while waiting.
- When the AI response includes edit suggestions, an **Apply Changes** button is visible.
- Clicking **Apply Changes** triggers the WebViewer content-edit flow and the PDF updates.
- The button is disabled after changes are applied to prevent double-application.
