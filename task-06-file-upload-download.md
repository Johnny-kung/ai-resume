# Task 06 — File Upload and PDF Download

## Goal
Allow users to upload their own resume PDF and download the modified version after AI edits have been applied.

## Dependencies
- Task 02 complete (WebViewer initialized)
- Task 05 complete (AI edits can be applied)

## Files to Create / Modify
```
client/src/
├── components/
│   ├── UploadButton.jsx     # File input that loads a PDF into WebViewer
│   └── DownloadButton.jsx   # Triggers PDF export from WebViewer
└── App.jsx                  # Add Upload and Download buttons to toolbar
```

## Steps

### 1. Upload Button — `components/UploadButton.jsx`
- Renders a styled `<input type="file" accept=".pdf" />` (hidden) triggered by a visible button.
- On file selection:
  1. Get the WebViewer instance from the Zustand store.
  2. Call `instance.loadDocument(file)` with the selected `File` object.
  3. Reset the chat history in the Zustand chat store so the conversation starts fresh for the new document.

```jsx
function UploadButton() {
  const instance = useViewerStore((s) => s.instance);
  const fileRef = useRef(null);

  function handleChange(e) {
    const file = e.target.files[0];
    if (file && instance) {
      instance.loadDocument(file);
      useChatStore.getState().clearMessages(); // add clearMessages to chat store
    }
  }

  return (
    <>
      <input ref={fileRef} type="file" accept=".pdf" hidden onChange={handleChange} />
      <button onClick={() => fileRef.current.click()}>Upload Resume</button>
    </>
  );
}
```

### 2. Download Button — `components/DownloadButton.jsx`
- Calls `instance.UI.downloadPDF()` which triggers a browser download of the current (potentially edited) document.
- Optionally pass `{ filename: 'my-resume.pdf' }` to set the download filename.

```jsx
function DownloadButton() {
  const instance = useViewerStore((s) => s.instance);

  function handleDownload() {
    if (instance) {
      instance.UI.downloadPDF({ filename: 'my-resume.pdf' });
    }
  }

  return <button onClick={handleDownload}>Download PDF</button>;
}
```

### 3. Update Chat Store — add `clearMessages`
Add the following action to `store/chatStore.js`:
```js
clearMessages: () => set({ messages: [] }),
```

### 4. Toolbar Layout — `App.jsx`
Add a top toolbar row above the two-panel layout:

```jsx
<div style={{ display: 'flex', flexDirection: 'column', height: '100vh' }}>
  {/* Top toolbar */}
  <div style={{ display: 'flex', gap: 8, padding: 8, borderBottom: '1px solid #ddd' }}>
    <UploadButton />
    <DownloadButton />
  </div>

  {/* Main panels */}
  <div style={{ display: 'flex', flex: 1 }}>
    <div style={{ flex: 2 }}>
      <PDFViewer initialDoc="/templates/resume-template.pdf" />
    </div>
    <div style={{ flex: 1 }}>
      <ChatPanel />
    </div>
  </div>
</div>
```

### 5. Edge Cases
| Scenario | Handling |
|---|---|
| User uploads a non-PDF file | `accept=".pdf"` filter on the input; show an alert if bypassed |
| Upload before WebViewer is ready | Disable the button until `instance` is set in the store |
| Download before any edits | Still works — downloads the current document unchanged |
| Large PDF upload | WebViewer handles it natively; no special handling needed |

## Acceptance Criteria
- User can upload any PDF and it loads into the WebViewer viewer, replacing the default template.
- Uploading a new file clears the chat history.
- Clicking **Download PDF** downloads the current document (including any AI-applied edits) to the user's machine.
- Upload button is disabled (or shows a loading state) until the WebViewer instance is initialized.
