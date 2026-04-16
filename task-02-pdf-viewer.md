# Task 02 — PDF Viewer with WebViewer Integration

## Goal
Embed the Apryse WebViewer in the React app and enable Content Editing mode so the PDF can be programmatically modified.

## Dependencies
- Task 01 complete (WebViewer assets in `client/public/webviewer/`)

## Files to Create / Modify
```
client/src/
├── components/
│   └── PDFViewer.jsx        # WebViewer container component
├── hooks/
│   └── useWebViewer.js      # WebViewer init + content-edit helpers
└── store/
    └── viewerStore.js       # Zustand store slice for viewer state
```

## Steps

### 1. Zustand Store — `store/viewerStore.js`
Create a Zustand store slice that holds:
- `instance` — the initialized WebViewer instance (null until ready)
- `setInstance(instance)` — setter

```js
// store/viewerStore.js
import { create } from 'zustand';

const useViewerStore = create((set) => ({
  instance: null,
  setInstance: (instance) => set({ instance }),
}));

export default useViewerStore;
```

### 2. WebViewer Hook — `hooks/useWebViewer.js`
- Accepts a `containerRef` (DOM element ref) and an initial document path.
- Initializes WebViewer from `public/webviewer/`.
- Enables the **ContentEdit** feature flag.
- Stores the instance in the Zustand store.
- Exposes an `applyEdits(edits)` helper that iterates over an array of `{ find, replace }` objects and updates the PDF content blocks via `ContentEdit.updateDocumentContent()`.

Key WebViewer APIs:
| Purpose | API |
|---|---|
| Initialize viewer | `WebViewer({ path, initialDoc }, container)` |
| Enable content edit | `instance.UI.enableFeatures([instance.UI.Feature.ContentEdit])` |
| Get content edit manager | `instance.Core.ContentEdit` |
| Edit a content block | `ContentEdit.updateDocumentContent(block, newText)` |
| Search text blocks | Iterate `PDFNet` page content or use annotation search |

### 3. PDFViewer Component — `components/PDFViewer.jsx`
- Renders a full-height `div` as the WebViewer mount point.
- Calls `useWebViewer` with the container ref.
- Accepts an optional `initialDoc` prop (URL or `File` object).

```jsx
// components/PDFViewer.jsx
import { useRef, useEffect } from 'react';
import useWebViewer from '../hooks/useWebViewer';

export default function PDFViewer({ initialDoc }) {
  const containerRef = useRef(null);
  useWebViewer(containerRef, initialDoc);

  return (
    <div
      ref={containerRef}
      style={{ height: '100vh', width: '100%' }}
    />
  );
}
```

### 4. App Layout — `App.jsx`
- Render `PDFViewer` on the left (two-thirds width).
- Reserve the right third for the `ChatPanel` (Task 03).

```jsx
<div style={{ display: 'flex', height: '100vh' }}>
  <div style={{ flex: 2 }}>
    <PDFViewer initialDoc="/templates/resume-template.pdf" />
  </div>
  <div style={{ flex: 1 }}>
    {/* ChatPanel goes here — Task 03 */}
  </div>
</div>
```

### 5. Resume Template
- Add a starter PDF resume template at `client/public/templates/resume-template.pdf` so users have something to edit out of the box.

## Acceptance Criteria
- The app opens with a resume PDF visible in WebViewer.
- Content Editing mode is enabled (toolbar shows text-edit tool).
- `useWebViewer` exports `applyEdits([{ find, replace }])` that can be called from the browser console to test text replacement in the PDF.
- The WebViewer instance is accessible via the Zustand store (`useViewerStore.getState().instance`).
