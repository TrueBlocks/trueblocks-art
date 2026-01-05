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

## 2.1. Yarn Command Location (ZERO TOLERANCE)

⛔ **NEVER RUN YARN FROM THE FRONTEND FOLDER. EVER.**

This is a HARD RULE with NO EXCEPTIONS:
- **NEVER** use `yarn --cwd frontend ...` or `yarn --cwd ./frontend ...`
- **NEVER** use `yarn --cwd /path/to/frontend ...`
- **ALWAYS** run yarn from the submodule root (e.g., `/works`)
- **ALWAYS** use `yarn lint`, `yarn test`, `yarn type-check` directly from root

```fish
# ABSOLUTELY FORBIDDEN — NEVER DO THIS
yarn --cwd frontend lint           # ❌ WRONG
yarn --cwd ./frontend type-check   # ❌ WRONG
yarn --cwd /path/to/frontend build # ❌ WRONG

# CORRECT — Always from submodule root
cd /path/to/works; and yarn lint       # ✅ RIGHT
cd /path/to/works; and yarn type-check # ✅ RIGHT
cd /path/to/works; and yarn build      # ✅ RIGHT
```

**Why this matters:**
1. The submodule root has the correct package.json with all scripts
2. Using `--cwd frontend` bypasses root-level configuration
3. Creates inconsistent behavior between agent and user

---

## 3. Critical Workflow

- **DO NOT RUN YARN COMMANDS** unless explicitly requested by the user
- **EXCEPTION**: Validation commands (`yarn lint; and yarn test`) acceptable after major changes
- **Read file contents first** before editing — files change between requests
- **After backend changes**: Run `wails generate module` to update TypeScript bindings
- **File deletion**: Use `rm -f` for files, `rm -R` for folders, ask confirmation first
- **Context updates**: When user renames/refactors, immediately update mental model

---

## 3.1. Wails Bindings (ZERO TOLERANCE)

- **NEVER edit files in `./frontend/wailsjs/`** — these are auto-generated
- Only regenerate by running `wails generate module` from the project root
- After any Go backend changes that affect exported functions, run:
  ```fish
  wails generate module
  ```

---

## 4. Directory Discipline (ZERO TOLERANCE)

⛔ **NEVER USE `cd` TO CHANGE INTO A SUBFOLDER. EVER.**

This is a HARD RULE with NO EXCEPTIONS:
- **NEVER** write `cd frontend` or `cd` into any subfolder
- **NEVER** run commands from inside subfolders
- **ALWAYS** run commands from the submodule root using `--prefix` or absolute paths

```fish
# ABSOLUTELY FORBIDDEN — NEVER DO THIS
cd frontend; and yarn build    # ❌ WRONG
cd frontend                    # ❌ WRONG
pushd frontend                 # ❌ WRONG

# CORRECT — Always use --prefix from submodule root
yarn --cwd frontend build      # ✅ RIGHT
yarn --cwd ./frontend build    # ✅ RIGHT
```

**Why this matters:**
1. Agents lose track of current directory between requests
2. Subsequent commands fail silently or in wrong location
3. Creates confusion and broken state

**If you catch yourself about to type `cd`:** STOP. Use `--cwd` or `--prefix` instead.

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

- **DRY (Don't Repeat Yourself)** — ZERO TOLERANCE
  - If you write the same logic twice, extract it immediately
  - Before implementing: "Does this pattern exist elsewhere? Should I extract a shared utility?"
  - Shared utilities go in `@/utils`, shared components in `@/components`, shared hooks in `@/hooks`
  - When adding features to one page/component, check if others need the same pattern
- **No over-engineering**: Simple, boring code that works
- **STOP and THINK**: "What's the simplest solution?"
- **No `any` in TypeScript** — always use specific types
- **No comments in production code** — only for TODO items
- **No commented-out code** — delete it
- Only comment *why*, never *what*
- **No lint suppressions** — never use `eslint-disable`, `@ts-ignore`, `@ts-expect-error`, or `nolint`
  - Fix the underlying issue instead of suppressing the warning
  - If lint rules conflict with valid code patterns, restructure the code

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

## 11. Go File Creation (CRITICAL BUG PREVENTION)

⚠️ **KNOWN ISSUE**: When creating `.go` files, there is a recurring bug where files get corrupted with:
- Duplicate `package` declarations at the top
- Many blank lines
- All remaining code collapsed onto a single line

**PREVENTION RULES:**

1. **Create ONE Go file at a time** — never batch multiple Go file creations in parallel
2. **Immediately verify** after creating any Go file by running `go build ./path/to/package/...`
3. **If verification fails**, delete the file with rm -f and recreate it from scratch
4. **Keep content simple** — avoid complex string escaping or special characters
5. **After creating a Go file**, read it back to confirm it's correctly formatted before moving on

**Recovery procedure if corruption occurs:**
```fish
rm -f path/to/corrupted.go
# Then recreate the file fresh
```

---

## 12. Testing

- Use **Vitest** for all tests, never Jest
- Do not run tests during implementation — user will run and report
- Check existing mocks before creating new ones

---

## 13. VS Code Problems Reset

When VS Code shows stale errors:
```
Cmd+Shift+P → "TypeScript: Restart TS Server"
Cmd+Shift+P → "Developer: Reload Window"
```

---

## 14. Git

- Commit early and often
- Clear messages: `feat:`, `fix:`, `refactor:`, `docs:`
- Never commit `node_modules/`, `build/`, or `.db` files

---

*See submodule-specific `.github/copilot-instructions.md` for project-specific rules.*
