# Leaning-Go-Full-Stack-Application-with-Vue

A full-stack web application for learning Go: a **Go** backend serving a JSON API and a **Vue 3 + TypeScript** frontend (built with **Vite**, styled with **Bootstrap**).

## Tech stack

| Layer    | Technology                                               |
| -------- | -------------------------------------------------------- |
| Backend  | Go (standard library `net/http`)                         |
| Frontend | Vue 3, TypeScript, Vite, Bootstrap 5, Pinia, Vue Router, Axios |
| Testing  | `go test` (backend), Vitest + Vue Test Utils + jsdom (frontend) |

## Prerequisites

- [Go](https://go.dev/dl/) 1.22 or newer (`go version`)
- [Node.js](https://nodejs.org/) 20 or newer (`node -v`)
- Git

## Setup steps

All commands are PowerShell, run from the repository root.

### 1. Backend (Go)

```powershell
New-Item -ItemType Directory -Force -Path `
  backend\cmd\server, backend\internal\handlers, backend\internal\middleware, `
  backend\internal\models, backend\internal\store, backend\internal\config, `
  backend\data, backend\tests | Out-Null

Set-Location backend
go mod init github.com/Ollie-S23/Leaning-Go-Full-Stack-Application-with-Vue/backend
New-Item cmd\server\main.go -ItemType File | Out-Null
Set-Location ..
```

Add a health-check endpoint to `backend/cmd/server/main.go`:

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("GET /api/health", func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Type", "application/json")
		json.NewEncoder(w).Encode(map[string]string{"status": "ok"})
	})

	log.Println("listening on :8080")
	log.Fatal(http.ListenAndServe(":8080", mux))
}
```

Copy the sample data files into `backend/data/`: `vehicles.json`, `modules.json`, `operations.json`, `workflows.json`, `telemetry_updates.json`.

### 2. Frontend (Vue + Vite + TypeScript + Bootstrap)

```powershell
npm create vite@latest frontend -- --template vue-ts
Set-Location frontend
npm install
npm install bootstrap @popperjs/core vue-router pinia axios
npm install -D vitest @vue/test-utils jsdom
Set-Location ..
```

(Answer **No** when Vite asks whether to install and start now.)

Import Bootstrap in `frontend/src/main.ts`:

```ts
import 'bootstrap/dist/css/bootstrap.min.css'
import 'bootstrap'
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'

createApp(App).mount('#app')
```

Configure the dev proxy and the test environment in `frontend/vite.config.ts`:

```ts
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: { proxy: { '/api': 'http://localhost:8080' } },
  test: { environment: 'jsdom' },
})
```

Add a test script to `frontend/package.json`:

```json
"scripts": {
  "dev": "vite",
  "build": "vue-tsc -b && vite build",
  "preview": "vite preview",
  "test": "vitest run --passWithNoTests"
}
```

### 3. Tooling and scripts (optional)

```powershell
go install golang.org/x/vuln/cmd/govulncheck@latest
go install github.com/securego/gosec/v2/cmd/gosec@latest
go install honnef.co/go/tools/cmd/staticcheck@latest
winget install ezwinports.make
```

The root `Makefile` provides these targets: `dev-backend`, `dev-frontend`, `test`, `lint`, `security` and `build`.

### 4. Git ignore

`.gitignore` ignores build output (`bin/`), editor settings (`.vscode/`), environment files (`.env`) and the usual Go artifacts. The Vite template adds its own `frontend/.gitignore` for `node_modules` and `dist`.

## Running the project

Start the backend and the frontend in two terminals:

```powershell
# Terminal 1 - Go API on :8080
Set-Location backend
go run ./cmd/server

# Terminal 2 - Vue dev server on :5173
Set-Location frontend
npm run dev
```

Or, with `make` installed: `make dev-backend` and `make dev-frontend`.

Verify the stack end to end (the request goes through the Vite proxy to Go):

```powershell
Invoke-RestMethod http://localhost:5173/api/health
# status
# ------
# ok
```

Then open <http://localhost:5173>.

## Testing and checks

```powershell
# Backend
Set-Location backend
go test ./... -cover
go vet ./...

# Frontend
Set-Location ../frontend
npm test
npm run build        # type-checks with vue-tsc, then builds

# Security (from the repo root)
Set-Location ../backend; govulncheck ./...; gosec ./...
Set-Location ../frontend; npm audit --omit=dev
```

## Project structure

```
.
├── .gitignore
├── .vscode/
├── Makefile
├── README.md
├── backend/
│   ├── go.mod
│   ├── cmd/server/main.go
│   ├── data/                 # JSON sample data
│   ├── internal/
│   │   ├── config/
│   │   ├── handlers/
│   │   ├── middleware/
│   │   ├── models/
│   │   └── store/
│   └── tests/
└── frontend/
    ├── package.json
    ├── vite.config.ts
    ├── tsconfig*.json
    └── src/
```

## Next steps

- Define Go structs for the JSON data files and load them at startup.
- Add API endpoints under `/api` and matching TypeScript types in the frontend.
- Add tests alongside each new package and component.

## References

- [Go module layout](https://go.dev/doc/modules/layout)
- [Go 1.22 routing enhancements](https://go.dev/blog/routing-enhancements)
- [Vite guide](https://vite.dev/guide/) and [server proxy](https://vite.dev/config/server-options.html#server-proxy)
- [Vue with TypeScript](https://vuejs.org/guide/typescript/overview.html)
- [Bootstrap with Vite](https://getbootstrap.com/docs/5.3/getting-started/vite/)
- [Vitest config](https://vitest.dev/config/)
