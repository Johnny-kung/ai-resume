# Task 01 — Project Scaffolding

## Goal
Set up the monorepo structure with a React + Vite frontend and a Golang backend.

## Directory Structure
```
ai-resume/
├── client/                  # React + Vite frontend
│   ├── public/
│   │   └── webviewer/       # Apryse WebViewer static lib assets (copied manually)
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── store/
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
├── server/                  # Golang backend
│   ├── main.go
│   ├── go.mod
│   └── go.sum
├── .env                     # Shared env vars (API keys, ports)
└── README.md
```

## Steps

### 1. Frontend — React + Vite
```bash
npm create vite@latest client -- --template react
cd client
npm install
```

Install core dependencies:
```bash
npm install @pdftron/webviewer zustand axios
```

### 2. Copy WebViewer Static Assets
- Purchase / obtain an Apryse WebViewer license.
- Copy the `lib` folder from the WebViewer npm package into `client/public/webviewer/`:
```bash
cp -r node_modules/@pdftron/webviewer/public client/public/webviewer
```

### 3. Backend — Golang
```bash
mkdir server && cd server
go mod init github.com/<your-org>/ai-resume/server
```

Install dependencies:
```bash
go get github.com/gin-gonic/gin
go get github.com/gin-contrib/cors
go get github.com/sashabaranov/go-openai
go get github.com/joho/godotenv
```

Create `server/main.go` with a basic Gin server that:
- Loads `.env`
- Enables CORS for the Vite dev server (`http://localhost:5173`)
- Registers `/api/chat` route placeholder
- Listens on `:8080`

### 4. Environment Variables (`.env`)
```
OPENAI_API_KEY=sk-...
SERVER_PORT=8080
VITE_API_BASE_URL=http://localhost:8080
```

## Acceptance Criteria
- `npm run dev` inside `client/` starts the Vite dev server on port 5173.
- `go run main.go` inside `server/` starts the Gin server on port 8080.
- CORS is configured so the frontend can call the backend without errors.
- WebViewer static assets are present at `client/public/webviewer/`.
