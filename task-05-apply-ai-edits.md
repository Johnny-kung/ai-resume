# Task 05 — Apply AI Edits to PDF via WebViewer Content Editing

## Goal
Implement the `applyEdits` function that takes the structured edit list returned by the AI and uses the Apryse WebViewer Content Editing API to find and replace text directly inside the PDF document.

## Dependencies
- Task 02 complete (WebViewer initialized, Content Editing feature enabled)
- Task 03 complete (Chat panel calls `applyEdits` on button click)
- Task 04 complete (backend returns `edits` array)

## Files to Modify
```
client/src/
└── hooks/
    └── useWebViewer.js      # Add applyEdits implementation
```

## Background: WebViewer Content Editing API

WebViewer's Content Edit mode works with **content edit placeholders** — objects representing editable text regions on each PDF page. The flow to modify text is:

1. Get all content edit placeholders for a page.
2. For each placeholder, retrieve its current text.
3. If the text contains the target string, call `ContentEdit.updateDocumentContent(placeholder, newText)`.

Key APIs:
| Step | API |
|---|---|
| Access content edit module | `instance.Core.ContentEdit` |
| Get placeholders for a page | `ContentEdit.getContentEditPlaceholders(pageNumber)` |
| Get placeholder text | `placeholder.getContents()` |
| Update placeholder text | `ContentEdit.updateDocumentContent(placeholder, newText)` |
| Get page count | `documentViewer.getPageCount()` |

## Steps

### 1. Implement `applyEdits` in `hooks/useWebViewer.js`

```js
/**
 * Apply a list of find-and-replace edits to the loaded PDF.
 * @param {Array<{find: string, replace: string}>} edits
 */
async function applyEdits(edits) {
  const instance = useViewerStore.getState().instance;
  if (!instance || !edits?.length) return;

  const { documentViewer, ContentEdit } = instance.Core;
  const doc = documentViewer.getDocument();
  const pageCount = documentViewer.getPageCount();

  for (let pageNum = 1; pageNum <= pageCount; pageNum++) {
    const placeholders = await ContentEdit.getContentEditPlaceholders(pageNum);

    for (const placeholder of placeholders) {
      const currentText = placeholder.getContents();

      for (const { find, replace } of edits) {
        if (currentText.includes(find)) {
          const updatedText = currentText.replaceAll(find, replace);
          await ContentEdit.updateDocumentContent(placeholder, updatedText);
          break; // placeholder updated; move to next placeholder
        }
      }
    }
  }
}
```

### 2. Expose `applyEdits` via the Zustand Viewer Store

Update `store/viewerStore.js` to also hold the `applyEdits` function so `ChatPanel` can call it without prop drilling:

```js
const useViewerStore = create((set, get) => ({
  instance: null,
  applyEdits: null,
  setInstance: (instance) => set({ instance }),
  setApplyEdits: (fn) => set({ applyEdits: fn }),
}));
```

In `useWebViewer.js`, after WebViewer initializes:
```js
useViewerStore.getState().setApplyEdits(applyEdits);
```

In `ChatPanel.jsx`, call it as:
```js
const applyEdits = useViewerStore((s) => s.applyEdits);
// ...
await applyEdits(message.edits);
```

### 3. Error Handling
- If a `find` string is not found in any page placeholder, log a warning and skip (do not throw).
- Wrap the entire loop in try/catch; on failure surface a toast/alert to the user.

### 4. Visual Feedback
- While `applyEdits` is running, show a loading indicator in the chat panel (set `isLoading = true` in the chat store).
- After completion, show a success toast: "Resume updated successfully".

## Acceptance Criteria
- Calling `applyEdits([{ find: 'old text', replace: 'new text' }])` modifies the matching text block in the visible PDF.
- All pages are scanned; multiple edits in one call are all applied.
- If no match is found for a given `find` string, the function continues processing remaining edits without crashing.
- The PDF viewer reflects changes immediately after `applyEdits` resolves (no manual refresh needed).
