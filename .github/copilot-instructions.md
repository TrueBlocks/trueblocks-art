# AI Agent Instructions (Shared)

> These rules apply to ALL submodules in the trueblocks-art mono-repo.
> Each submodule may have additional project-specific rules in its own `.github/copilot-instructions.md`.

---

## 1. Shell Environment

- **fish shell ONLY** — Never use bash syntax.
- No `&&` chaining. Use `; and` or separate commands.
- No `export VAR=value`. Use `set -x VAR value`.
- No `$(command)`. Use `(command)` for command substitution.
- No `[[` conditionals. Use `test` or `[`.
- Never use heredoc.

```fish
# WRONG
cd frontend && npm install
export GOPATH=$HOME/go

# RIGHT
cd frontend; and npm install
set -x GOPATH $HOME/go
```

---

## 2. Package Management (ZERO TOLERANCE)

- **YARN ONLY** — Never use `npm` or `npx`
- All commands run from submodule root: `yarn start`, `yarn build`, `yarn test`, `yarn lint`

---

## 3. Critical Workflow

- **DO NOT RUN YARN COMMANDS** unless explicitly requested by the user
- **EXCEPTION**: Validation commands (`yarn lint; and yarn test`) acceptable after major changes
- **Read file contents first** before editing — files change between requests
- **After backend changes**: Run `wails generate module` to update TypeScript bindings
- **File deletion**: Use `rm -f` for files, `rm -R` for folders, ask confirmation first
- **Context updates**: When user renames/refactors, immediately update mental model

---

## 4. Directory Discipline

- **Never cd into a subfolder and stay there.**
- If you must cd, immediately cd back when done.
- Prefer absolute paths or path arguments instead of cd.

```fish
# WRONG
cd frontend
npm install
npm run build
# (forgot to cd back)

# RIGHT
npm --prefix frontend install
npm --prefix frontend run build

# OR
cd frontend; npm install; cd ..
```

---

## 5. Step-by-Step Mode

When user says "go into step-by-step mode":

🚫 **Never Run:**
- `yarn lint`
- `yarn test`
- `yarn start`

🛑 **Stop Between Steps:**
- Never run amok or jump ahead
- Stop after each step for review and approval
- Wait for "go ahead" before proceeding

📋 **Planning Process:**
- Show what you want to do WITHOUT modifying code first
- Explain WHY you want to make each change
- Wait for approval before making any code changes

🔒 **PERSISTENCE RULE:**
- **ONCE IN STEP-BY-STEP MODE, STAY THERE INDEFINITELY**
- Do NOT exit unless explicitly told "exit step-by-step mode"
- Every action must be approved individually

---

## 6. Design Mode

When user says "go into design mode":

🎨 **MODE IDENTIFIER (REQUIRED):**
Every response MUST start with:
```
🎨 DESIGN MODE | [Can: discuss/analyze] [Cannot: implement/modify code]
```

📋 **Rules:**
- NO full codebase scans upfront
- Discussion-focused for architectural analysis
- Read files just-in-time before discussing specifics
- NO code modifications, builds, tests, or implementations

🚫 **FORBIDDEN PHRASES:**
- "I'll implement...", "Let me create...", "I'll modify...", "I'll add..."

If caught: "I cannot implement code changes while in design mode."

🔒 **NO MODIFICATION RULE:**
- Stay in design mode indefinitely until explicit exit
- Even if user asks to implement: respond "I cannot implement while in design mode"

---

## 7. Mode Switching Rules (CRITICAL)

- **MUTUALLY EXCLUSIVE**: Never be in both design mode and step-by-step mode
- **NO SELF-INITIATED MODE EXITS**: Never exit any mode on your own
- **Only two ways to exit:**
  1. Explicit command: "exit [mode-name] mode"
  2. Command to enter other mode: "go into [other-mode] mode"
- **Mode persistence**: Once in a mode, stay there across all requests

---

## 8. Code Quality Principles

- **No over-engineering**: Simple, boring code that works
- **STOP and THINK**: "What's the simplest solution?"
- **No `any` in TypeScript** — always use specific types
- **No comments in production code** — only for TODO items
- **No commented-out code** — delete it
- Only comment *why*, never *what*

---

## 9. Collaboration Protocol

- **Ask early, ask often**: When complexity creeps in, stop and discuss
- **Own mistakes**: Don't blame "someone" — broken code is your responsibility
- **Stop conditions**: Test failures, lint errors, unclear requirements — stop and report

---

## 10. Language Constraints

### Go Backend
- **Never use Python** — not for scripts, not for one-liners, not for "quick" tools
- All backend code, CLI tools, and utilities must be Go
- Use `go run` for quick scripts if needed

### TypeScript/React Frontend
- React 18 + TypeScript 5 + Mantine 7
- No JavaScript files — always `.ts` or `.tsx`
- No class components — functional components only
- No React imports (implicitly available)
- Use `Log` from `@utils` instead of console.log

---

## 11. Testing

- Use **Vitest** for all tests, never Jest
- Do not run tests during implementation — user will run and report
- Check existing mocks before creating new ones

---

## 12. VS Code Problems Reset

When VS Code shows stale errors:
```
Cmd+Shift+P → "TypeScript: Restart TS Server"
Cmd+Shift+P → "Developer: Reload Window"
```

---

## 13. Git

- Commit early and often
- Clear messages: `feat:`, `fix:`, `refactor:`, `docs:`
- Never commit `node_modules/`, `build/`, or `.db` files

---

*See submodule-specific `.github/copilot-instructions.md` for project-specific rules.*
