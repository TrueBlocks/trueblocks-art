# Creating New Apps in trueblocks-art

This guide explains how to create a new desktop application in the trueblocks-art mono-repo.

---

## Prerequisites

- Go 1.21+
- Node.js 20+
- Wails v2: `go install github.com/wailsapp/wails/v2/cmd/wails@latest`
- LibreOffice (for PDF generation)

---

## Step 1: Create the Submodule

```fish
# From the mono-repo root
cd ~/Development/trueblocks-art

# Create new repo on GitHub first, then:
git submodule add git@github.com:TrueBlocks/trueblocks-{appname}.git
cd trueblocks-{appname}
```

---

## Step 2: Initialize Wails Project

```fish
# From inside the new submodule
wails init -n {appname} -t react-ts .

# This creates:
# - main.go, app.go
# - wails.json
# - frontend/ (React + TypeScript + Vite)
```

---

## Step 3: Add CLAUDE.md

Copy the AI instructions file to the repo root:

```fish
cp ../trueblocks-works/CLAUDE.md .
```

Edit the file to update app-specific details if needed.

---

## Step 4: Create Design Folder

```fish
mkdir design
```

Create these essential specification documents:

| File | Purpose |
|------|---------|
| `specification.md` | Master index linking all docs |
| `01-data-model.md` | SQLite schema, field definitions, types |
| `02-relationships.md` | Foreign keys, cascades, junction tables |
| `03-business-logic.md` | Go backend functions, workflows |
| `04-value-lists.md` | Enums, dropdown options, status values |
| `07-file-management.md` | File paths, document handling |
| `09-app-state.md` | Persisted state, user preferences |

Optional but recommended:

| File | Purpose |
|------|---------|
| `06-layouts.md` | UI structure, page layouts, components |
| `10-error-handling.md` | Error patterns, logging |
| `13-search.md` | Full-text search with FTS5 |

---

## Step 5: Set Up Go Structure

```fish
mkdir -p internal/{db,state,fileops}
```

### internal/db/
- `db.go` — Database connection, migrations
- `works.go` — CRUD for works (or your main entity)
- `schema.sql` — SQLite schema

### internal/state/
- `persisted.go` — Saved preferences, window state
- `runtime.go` — In-memory app state

### internal/fileops/
- `paths.go` — Path generation, validation
- `pdf.go` — LibreOffice PDF generation

---

## Step 6: Set Up Frontend

```fish
yarn --cwd frontend add @mantine/core @mantine/hooks @tabler/icons-react react-router-dom
```

Then add yarn scripts to the root `package.json`:

```json
{
  "scripts": {
    "start": "wails dev",
    "build": "wails build -platform darwin/universal",
    "test": "yarn test:go && yarn test:frontend",
    "test:go": "go test ./...",
    "test:frontend": "yarn --cwd frontend test",
    "lint": "yarn --cwd frontend lint"
  }
}
```

### Folder Structure

```
frontend/src/
├── main.tsx          # Entry point
├── App.tsx           # Router + AppShell
├── components/       # Reusable UI components
│   ├── StatusBadge.tsx
│   └── ...
├── pages/            # Route pages
│   ├── CollectionsPage.tsx
│   └── ...
├── hooks/            # Custom React hooks
│   └── useKeyboardShortcuts.ts
└── types.ts          # TypeScript interfaces
```

---

## Step 7: Configure Wails Bindings

In `app.go`, export methods that React can call:

```go
type App struct {
    ctx context.Context
    db  *sql.DB
}

func (a *App) GetWorks() result.Result[[]Work] {
    works, err := db.GetAllWorks(a.db)
    if err != nil {
        return result.Fail[[]Work](err, "DB_ERROR")
    }
    return result.Ok(works)
}
```

Generate TypeScript bindings:

```fish
wails generate module
```

---

## Step 8: Run Development Server

```fish
yarn start
```

This runs `wails dev` which starts:
- Go backend with hot reload
- Vite dev server for frontend
- Opens app window

---

## Standard Patterns

### Result Type (Go)

```go
type Result[T any] struct {
    Success bool   `json:"success"`
    Data    T      `json:"data,omitempty"`
    Error   *Error `json:"error,omitempty"`
}

func Ok[T any](data T) Result[T] {
    return Result[T]{Success: true, Data: data}
}

func Fail[T any](err error, code string) Result[T] {
    return Result[T]{Success: false, Error: &Error{Code: code, Message: err.Error()}}
}
```

### Keyboard Shortcuts (React)

```typescript
import { useHotkeys } from '@mantine/hooks';

useHotkeys([
    ['mod+1', () => navigate('/')],
    ['mod+2', () => navigate('/works')],
    ['mod+k', () => openSearch()],
]);
```

### SQLite Connection

```go
import "modernc.org/sqlite"

// App data lives in ~/.local/share/trueblocks/<app-name>/
func getAppDataDir(appName string) string {
    home, _ := os.UserHomeDir()
    return filepath.Join(home, ".local", "share", "trueblocks", appName)
}

appDataDir := getAppDataDir("works")
os.MkdirAll(appDataDir, 0755)
db, err := sql.Open("sqlite", filepath.Join(appDataDir, "app.db"))
db.Exec("PRAGMA journal_mode=WAL")
db.Exec("PRAGMA foreign_keys=ON")
```

---

## Checklist for New App

- [ ] Submodule added to mono-repo
- [ ] Wails project initialized
- [ ] CLAUDE.md copied and customized
- [ ] Design docs created (at minimum: data model, business logic)
- [ ] Go internal/ structure set up
- [ ] Frontend dependencies installed via yarn
- [ ] Root package.json with yarn scripts
- [ ] TypeScript types defined
- [ ] At least one page renders
- [ ] Database connection works
- [ ] `yarn start` runs without errors

---

## Building for Release

```fish
yarn build
```

This runs `wails build -platform darwin/universal`.

Output: `build/bin/{AppName}.app`

For distribution:
1. Code sign with Developer ID
2. Notarize with Apple
3. Staple notarization ticket

---

## Reference

See `trueblocks-works` for a complete working example.

---

*Last updated: January 3, 2026*
