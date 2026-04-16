# Task 04 — Golang AI Backend

## Goal
Build a Golang/Gin HTTP server that receives chat messages plus the current resume text, calls OpenAI, and returns the AI reply along with a structured list of PDF edits when the AI decides changes should be applied.

## Dependencies
- Task 01 complete (Go module initialized, dependencies installed)

## Files to Create / Modify
```
server/
├── main.go                  # Gin server entry point, route registration
├── handlers/
│   └── chat.go              # POST /api/chat handler
├── services/
│   └── openai.go            # OpenAI client wrapper
├── models/
│   └── chat.go              # Request / response structs
└── .env                     # OPENAI_API_KEY, SERVER_PORT
```

## Steps

### 1. Data Models — `models/chat.go`
```go
package models

// Inbound request from frontend
type ChatRequest struct {
    Messages   []Message `json:"messages"`
    ResumeText string    `json:"resumeText"`
}

type Message struct {
    Role    string `json:"role"`    // "user" | "assistant"
    Content string `json:"content"`
}

// Outbound response to frontend
type ChatResponse struct {
    Reply string     `json:"reply"`
    Edits []EditItem `json:"edits,omitempty"`
}

type EditItem struct {
    Find    string `json:"find"`
    Replace string `json:"replace"`
}
```

### 2. OpenAI Service — `services/openai.go`
- Initialize the `go-openai` client from `OPENAI_API_KEY`.
- Define a `Chat(messages []models.Message, resumeText string) (*models.ChatResponse, error)` function.
- System prompt instructs the model to:
  - Act as a professional resume coach.
  - When the user confirms they want changes applied, respond with a JSON block inside the reply in the form:
    ```
    EDITS_JSON:
    [{"find":"old text","replace":"new text"}, ...]
    ```
  - Otherwise respond conversationally.
- After receiving the model reply, parse out the `EDITS_JSON:` block (if present) into `[]EditItem` and return both the cleaned reply text and the edits.

```go
// Pseudocode for parsing
const marker = "EDITS_JSON:"
if idx := strings.Index(reply, marker); idx != -1 {
    jsonStr := strings.TrimSpace(reply[idx+len(marker):])
    json.Unmarshal([]byte(jsonStr), &edits)
    reply = strings.TrimSpace(reply[:idx])
}
```

### 3. Chat Handler — `handlers/chat.go`
- Bind JSON body to `models.ChatRequest`.
- Call `services.Chat(req.Messages, req.ResumeText)`.
- Return `models.ChatResponse` as JSON.
- Return appropriate HTTP error codes on failure (400 for bad input, 502 if OpenAI fails).

```go
func ChatHandler(c *gin.Context) {
    var req models.ChatRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    resp, err := services.Chat(req.Messages, req.ResumeText)
    if err != nil {
        c.JSON(502, gin.H{"error": "AI service unavailable"})
        return
    }
    c.JSON(200, resp)
}
```

### 4. Server Entry Point — `main.go`
- Load `.env` with `godotenv`.
- Create Gin router.
- Add CORS middleware allowing `http://localhost:5173`.
- Register `POST /api/chat` → `handlers.ChatHandler`.
- Listen on `SERVER_PORT` (default `8080`).

### 5. System Prompt
```
You are an expert resume coach. The user will share their resume text and ask for improvements.
Discuss changes conversationally. When the user explicitly confirms they want the changes applied,
output the exact text replacements at the end of your message using this format:

EDITS_JSON:
[{"find":"<exact original text>","replace":"<new text>"}]

Only include EDITS_JSON when the user confirms. Keep "find" values verbatim from the resume.
```

## Running the Server
```bash
cd server
go run main.go
```

## Testing
Use `curl` or a REST client:
```bash
curl -X POST http://localhost:8080/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role":"user","content":"Make my summary more impactful"}],
    "resumeText": "Summary: I am a software engineer."
  }'
```

## Acceptance Criteria
- `POST /api/chat` returns `{ reply, edits }` where `edits` is populated only when the AI signals changes.
- Returns 400 on malformed request body.
- Returns 502 if the OpenAI call fails.
- CORS headers allow requests from `http://localhost:5173`.
- API key is read from environment and never hard-coded.
