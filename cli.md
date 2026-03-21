# CLI SYSTEM PRINCIPLE CHECKLISTS

**Reference Implementation:** This document uses `news-cli` — a Node.js CLI tool that aggregates news from Reddit and Substack — as the reference for all principles and examples.

---

## 🎯 IO BOUNDARY CHECKLIST

### Principle:
The CLI must establish clear boundaries at stdin, stdout, and stderr. All external communication flows through these three channels only.

```
  stdin  ──► [ CLI Process ] ──► stdout   (normal output, data)
                              ──► stderr   (errors, diagnostics)
```

### Checklist

- [ ] **Does all normal output go to stdout, not stderr?**
  - *Why this is important:* stdout is expected to contain the actual result (e.g., formatted posts). If you send data to stderr, piping to another tool will break (`news reddit update | grep`).
  - *news-cli example:* All formatted posts are printed to stdout via `console.log()`. Errors go to `console.error()` in a `try/catch`.

- [ ] **Are all errors sent to stderr?**
  - *Why this is important:* Allows piping (`news update | grep`) to work correctly — error messages won't pollute the data stream.
  - *news-cli example:* Network failures are caught in `try/catch` and printed with `spinner.fail()`, which writes to stderr.

- [ ] **Is the stdout stream composable with other tools?**
  - *Why this is important:* A well-designed CLI's output can become another tool's input. This is the Unix philosophy.
  - *news-cli example:* Running `news reddit update` produces one post per boxed section — human-readable but also machine-readable (could be parsed by another tool).

- [ ] **Is stdin being used when interactive input is needed?**
  - *Why this is important:* Allows the tool to be part of a pipe chain, not just standalone. Example: `cat subscriptions.txt | news import`.
  - *news-cli example:* Currently, news-cli doesn't read from stdin, only from the filesystem. This is fine for a non-interactive tool.

- [ ] **Are there any hardcoded terminal writes outside stdout/stderr? (If yes → FAIL)**
  - *Why this is important:* Writing directly to the terminal (e.g., via escape sequences not routed through stdout) breaks piping and makes the tool incompatible with non-terminal environments.
  - *news-cli example:* chalk and ora both write through normal stdout, not directly to `/dev/tty`.

### How this enforces the principle

- Ensures the CLI is **composable** with other Unix tools
- Prevents **data leakage** (errors polluting output)
- Enables **scripting and automation**

---

## 🔄 ADAPTER LAYER CHECKLIST

### Principle:
The CLI is a translation layer between human intent (natural language commands) and machine action (API calls, file operations). It adapts between two incompatible worlds.

```
  Human language          CLI Adapter             Machine language
  ──────────────          ───────────             ────────────────
  "subscribe to           $ news reddit       →   POST /api/subscriptions
   worldnews"             subscribe worldnews     { subreddit: "worldnews" }
```

### Checklist

- [ ] **Does the CLI parse human intent (arguments/flags) into machine actions?**
  - *Why this is important:* The parsing layer (commander) translates messy human input into structured, predictable commands.
  - *news-cli example:* `news reddit subscribe worldnews` is parsed by commander into `{ command: 'subscribe', subreddit: 'worldnews' }`.

- [ ] **Are command names intuitive and aligned with user mental models?**
  - *Why this is important:* If users have to guess or read docs constantly, adoption suffers. Command names should feel natural.
  - *news-cli example:* `subscribe`, `unsubscribe`, `update`, `list` match what users expect from a subscription manager.

- [ ] **Is argument/flag validation happening at the entry point?**
  - *Why this is important:* Failing early with a clear message beats failing deep in the logic with a cryptic error.
  - *news-cli example:* commander validates that `subscribe` receives exactly one argument (the subreddit name).

- [ ] **Are all transformations (human → machine) reversible or documented?**
  - *Why this is important:* If `news substack subscribe paulgraham` silently transforms to `https://paulgraham.substack.com`, the user should understand this happens.
  - *news-cli example:* `normalizeUrl()` in substack.js accepts both `"paulgraham"` and `"https://paulgraham.substack.com"` and normalizes to the full URL. This is documented in the help text.

- [ ] **Is the output rendering (machine → human) a pure translation?**
  - *Why this is important:* The output layer should never perform business logic. It only transforms data for display.
  - *news-cli example:* `formatPost()` takes a post object and returns a colored string. It does NOT fetch data, validate, or mutate state.

- [ ] **Are adapters isolated from business logic?**
  - *Why this is important:* If fetching from Reddit is tightly coupled to chalk colors, changing either one breaks both.
  - *news-cli example:* `fetchSubreddit()` is a pure data function. `formatPost()` is a pure rendering function. They are separate.

### How this enforces the principle

- Maintains **separation of concerns** (parsing → logic → rendering)
- Enables **reusability** (formatPost can be swapped without changing fetch logic)
- Supports **future UX changes** (could swap chalk/boxen for JSON output without touching business logic)

---

## 💾 PROCESS STATE CHECKLIST

### Principle:
A CLI process is stateless and ephemeral. It starts, runs, and dies. All state that must survive must live **outside** the process.

```
  Process 1            Process 2            Process 3
  ┌─────────┐          ┌─────────┐          ┌─────────┐
  │ news    │          │ news    │          │ news    │
  │ reddit  │          │ reddit  │          │ reddit  │
  │subscribe│          │subscribe│          │ update  │
  │worldnews│          │tech     │          │         │
  └────┬────┘          └────┬────┘          └────┬────┘
       │                    │                    │
       ▼                    ▼                    ▼
  ┌────────────────────────────────────────────────┐
  │  ~/.news-cli/subscriptions.json                │
  │  (the only thing that persists)                │
  └────────────────────────────────────────────────┘
```

### Checklist

- [ ] **Does the process maintain any global or module-level state? (If yes → question it)**
  - *Why this is important:* Global state makes the CLI non-idempotent — running the same command twice may produce different results.
  - *news-cli example:* Each invocation of `news` starts fresh. config.js doesn't cache anything globally; it reads from disk each time.

- [ ] **Is all persistent state stored outside the process?**
  - *Why this is important:* Ensures that state survives process death, restarts, and concurrent invocations.
  - *news-cli example:* Subscriptions are stored in `~/.news-cli/subscriptions.json`, not in memory.

- [ ] **Is state stored in a single, well-defined location?**
  - *Why this is important:* Prevents multiple copies of "truth" (one in RAM, one on disk) that can diverge.
  - *news-cli example:* `config.js` defines `CONFIG_DIR` and `CONFIG_FILE` in one place. Everything goes through those constants.

- [ ] **Can the process be killed mid-operation without corrupting state?**
  - *Why this is important:* Processes crash or get forcefully terminated. If you're writing a partial JSON file and the process dies, the next run should not fail.
  - *news-cli example:* config.js uses `fs.writeFileSync()` (atomic write). If a write is interrupted, the old file is still intact.

- [ ] **Is there a clear separation between ephemeral state (the current run) and persistent state (across runs)?**
  - *Why this is important:* Makes it obvious what data is safe to lose and what must be protected.
  - *news-cli example:* Fetched posts are printed and discarded. Subscriptions are written to disk. Clear boundary.

- [ ] **Can two processes safely run at the same time?**
  - *Why this is important:* Users may run the CLI in multiple terminal tabs. If both try to write subscriptions.json, one might corrupt it.
  - *news-cli example:* Currently, there's no file locking. If two processes write simultaneously, corruption is possible. This should be addressed with a lock file or atomic writes.

### How this enforces the principle

- Ensures **idempotence** (same command, same result)
- Prevents **global state pollution**
- Enables **safe process termination**
- Supports **concurrent invocations**

---

## 🎨 OUTPUT RENDERING CHECKLIST

### Principle:
The output layer is purely a rendering engine. It translates machine data into human-readable form. It contains zero business logic.

```
  POST OBJECT              FORMATTING              RENDERED OUTPUT
  ───────────              ──────────              ────────────────
  {                        formatPost()            ╭─────────────╮
    title: "...",     →    └─ chalk colors    →   │ Title...    │
    score: 4821,           └─ boxen border        │ metadata    │
    permalink: "/"         └─ newlines            │ url         │
  }                                                ╰─────────────╯
```

### Checklist

- [ ] **Does the output layer contain any data fetching, validation, or mutation? (If yes → FAIL)**
  - *Why this is important:* If rendering calls an API, changing the terminal colors risks breaking the API call.
  - *news-cli example:* `formatPost()` is a pure function. It takes a post, returns a string. No side effects.

- [ ] **Can the output format be swapped without changing business logic?**
  - *Why this is important:* Demonstrates that logic and rendering are truly separate.
  - *news-cli example:* You could replace chalk/boxen with plain console.log or JSON output without touching `fetchSubreddit()`.

- [ ] **Are colors/formatting driven by data, not hardcoded?**
  - *Why this is important:* Allows future changes (themes, accessibility, conditional formatting) without rewriting the whole layer.
  - *news-cli example:* Colors are applied consistently via chalk functions. Could be replaced with theme objects later.

- [ ] **Is the spinner/progress indication isolated from the data pipeline?**
  - *Why this is important:* A failure in the spinner library should not crash the data fetch.
  - *news-cli example:* `ora` is started/managed independently of the `fetch()` call. If ora fails, fetch can still complete.

- [ ] **Are error messages actionable?**
  - *Why this is important:* "Error" is useless. "Failed to fetch r/worldnews: HTTP 429 (Rate Limited)" tells the user what to do.
  - *news-cli example:* Error messages include the source and the HTTP status code, so users understand what went wrong.

- [ ] **Is the output deterministic (same input → same output)?**
  - *Why this is important:* Makes testing and debugging predictable.
  - *news-cli example:* Given the same post object, `formatPost()` always produces the same string.

### How this enforces the principle

- Maintains **separation of rendering from logic**
- Enables **output format flexibility**
- Ensures **composability** (output of one stage is input to the next)
- Supports **testing** (pure functions are easy to test)

---

## 🚨 ERROR CONTAINMENT CHECKLIST

### Principle:
Errors in one command or data source must not crash the entire CLI. Failures are local.

```
  subreddit 1: worldnews  ──► SUCCESS  ──► print posts
  subreddit 2: tech       ──► NETWORK  ──► spinner.fail()  ──► continue
                               ERROR         (does NOT crash)
  subreddit 3: programming ──► SUCCESS ──► print posts
```

### Checklist

- [ ] **Is every external API call wrapped in try/catch?**
  - *Why this is important:* Network calls can fail for many reasons (timeout, DNS failure, rate limit, server down). Uncaught errors crash the whole CLI.
  - *news-cli example:* Every `fetchSubreddit()` and `fetchSubstack()` call is wrapped in try/catch. Failures are caught and reported, not rethrown.

- [ ] **Are failures reported clearly without crashing?**
  - *Why this is important:* Users should see which sources worked and which failed, then continue.
  - *news-cli example:* A failed fetch prints `spinner.fail(chalk.red(...))` and the loop continues to the next subreddit.

- [ ] **Are error types distinguishable?**
  - *Why this is important:* A network timeout (try again) is different from a 403 Forbidden (check permissions). Users need to know what went wrong.
  - *news-cli example:* Currently, all errors are caught as generic errors. Could be improved by catching specific HTTP status codes.

- [ ] **Is the error context preserved (what were you doing when this failed)?**
  - *Why this is important:* "Failed to fetch" is useless. "Failed to fetch r/worldnews" tells the user exactly which source failed.
  - *news-cli example:* Error messages include the subreddit or Substack name in the spinner text.

- [ ] **Can the user recover from an error and retry?**
  - *Why this is important:* If a network blip causes one source to fail, the user should be able to re-run the command without losing work.
  - *news-cli example:* Running `news reddit update` again will retry all subreddits. No state is lost if one fails.

- [ ] **Are errors logged for debugging without exposing sensitive data?**
  - *Why this is important:* Developers need to see stack traces to fix bugs, but log files should never contain API keys or user data.
  - *news-cli example:* Error messages show HTTP status codes and response headers, but not request bodies or auth tokens.

### How this enforces the principle

- Prevents **cascading failures** (one bad source doesn't kill the whole run)
- Maintains **user trust** (errors are visible and actionable, not silent)
- Enables **resilience** (the CLI continues working even when external services fail)

---

## 🌐 EXTERNAL API BOUNDARY CHECKLIST

### Principle:
External APIs (Reddit, Substack) are outside your control. They are unreliable by default. Treat them as such.

```
  Your CLI            System Boundary          External API
  ────────            ────────────             ────────────
  fetchSubreddit()
  ├─ set User-Agent   ──────────────────►      Reddit servers
  ├─ await fetch()    ◄──────────────────      (may timeout,
  ├─ validate HTTP                             rate limit,
  └─ parse JSON                                or be down)
```

### Checklist

- [ ] **Are timeouts enforced on all external calls?**
  - *Why this is important:* If Reddit's API hangs, your fetch will hang forever, tying up the process.
  - *news-cli example:* Currently, there are no explicit timeouts. A timeout should be added: `fetch(url, { signal: AbortSignal.timeout(5000) })`.

- [ ] **Is the API response validated before use?**
  - *Why this is important:* Third-party APIs can change format, return unexpected fields, or return errors in an unusual way.
  - *news-cli example:* Currently, we assume `json.data.children` exists. If Reddit changes the format, the code will crash. Should validate first.

- [ ] **Is the User-Agent header set?**
  - *Why this is important:* Many APIs rate limit or block requests without a User-Agent. It's the CLI identifying itself politely.
  - *news-cli example:* `const USER_AGENT = 'news-cli/1.0.0'` is sent with every request to Reddit and Substack.

- [ ] **Are HTTP status codes checked before processing?**
  - *Why this is important:* A 429 (rate limited) or 503 (service down) is different from a 200 (OK). Treating them the same causes silent corruption.
  - *news-cli example:* `if (!res.ok) throw new Error(...)` checks the status code before parsing JSON.

- [ ] **Is rate limiting handled gracefully?**
  - *Why this is important:* If you hit a rate limit, you should wait and retry, not just fail.
  - *news-cli example:* Currently, rate limits result in an error message. Could be improved with exponential backoff retry.

- [ ] **Are API failures isolated from other data sources?**
  - *Why this is important:* If Reddit is down, Substack still works.
  - *news-cli example:* reddit.js and substack.js are independent. A failure in one doesn't affect the other.

- [ ] **Is the API contract documented?**
  - *Why this is important:* If the API changes, you need to know what changed. Having documented contracts helps.
  - *news-cli example:* The fetch functions have comments explaining what endpoints they hit and what the response shape is.

### How this enforces the principle

- Prevents **cascading API failures**
- Ensures **graceful degradation** (one API down ≠ whole CLI down)
- Enables **debugging** (clear error messages when APIs fail)
- Demonstrates **respect for external rate limits**

---

## ⚙️ CONFIGURATION MANAGEMENT CHECKLIST

### Principle:
Configuration is the bridge between user intent and CLI behavior. It must be stored durably, read safely, and modified atomically.

### Checklist

- [ ] **Is configuration stored in a well-known, documented location?**
  - *Why this is important:* Users need to know where to find and manually edit their config if needed.
  - *news-cli example:* Subscriptions are stored at `~/.news-cli/subscriptions.json`. This is printed to the user when they first subscribe.

- [ ] **Is the config location cross-platform compatible?**
  - *Why this is important:* Hardcoding `/home/user/.config/` fails on Windows. Use `os.homedir()` instead.
  - *news-cli example:* Uses `os.homedir()` to find the home directory, then stores under `~/.news-cli/` on all platforms.

- [ ] **Is the config file auto-created if missing?**
  - *Why this is important:* Users shouldn't have to manually create a config file. The CLI should handle first-run setup.
  - *news-cli example:* `ensureConfig()` creates the directory and file if they don't exist.

- [ ] **Is the config format human-readable?**
  - *Why this is important:* Allows power users to directly edit the file, and makes debugging easier.
  - *news-cli example:* Config is plain JSON with formatting (not minified), so it's easy to read.

- [ ] **Are writes atomic (all-or-nothing)?**
  - *Why this is important:* If the process dies mid-write, the old file should still be intact, not corrupted.
  - *news-cli example:* `fs.writeFileSync()` is atomic on most systems. For safety, could use write-to-temp-then-move pattern.

- [ ] **Is config read-then-mutate-then-write safe against concurrent processes?**
  - *Why this is important:* If two instances of the CLI modify config simultaneously, one might overwrite the other's changes.
  - *news-cli example:* Currently, there's no locking mechanism. This is a limitation worth noting.

- [ ] **Are invalid config values detected and reported?**
  - *Why this is important:* If a user hand-edits the JSON and makes a mistake, the CLI should fail with a clear error, not silent corruption.
  - *news-cli example:* Currently, parsing errors would just throw. Could add validation layer before parsing.

### How this enforces the principle

- Ensures **durability** (subscriptions survive restarts)
- Enables **transparency** (users can read and edit the file)
- Supports **portability** (works on all platforms)
- Maintains **consistency** (config is the single source of truth)

---

## 📦 DEPENDENCY MANAGEMENT CHECKLIST

### Principle:
Dependencies (npm packages) are external systems you don't own. Minimize them and choose wisely.

### Checklist

- [ ] **Is each dependency justified? (Could it be built in?)**
  - *Why this is important:* Every dependency is a potential security risk and maintenance burden.
  - *news-cli example:* Uses `chalk` (colors), `ora` (spinner), `boxen` (borders), `commander` (CLI parsing). Each solves a specific problem that would be tedious to build from scratch.

- [ ] **Are transitive dependencies (dependencies of dependencies) understood?**
  - *Why this is important:* Installing `package-a` might pull in 10 other packages. Security issues in any of them affect your CLI.
  - *news-cli example:* Use `npm ls` to audit the dependency tree and identify bloat.

- [ ] **Is the Node version constraint realistic?**
  - *Why this is important:* Claiming support for Node 12 when you use Node 18 features is misleading.
  - *news-cli example:* `"engines": { "node": ">=18.0.0" }` in package.json documents the minimum required Node version.

- [ ] **Are dependencies pinned to specific versions (or at least minor versions)?**
  - *Why this is important:* `^1.2.3` might upgrade to 1.9.0 tomorrow, which could introduce breaking changes.
  - *news-cli example:* package.json uses specific versions (`^5.3.0` means "at least 5.3.0, but not 6.0.0"), preventing major breaking changes.

- [ ] **Is the CLI packaged and published with dependencies locked?**
  - *Why this is important:* Users install your CLI expecting stable behavior, not random upgrades.
  - *news-cli example:* `npm publish` includes a `package-lock.json` to ensure reproducible installs.

### How this enforces the principle

- Reduces **attack surface** (fewer dependencies = fewer vulnerabilities)
- Maintains **stability** (locked versions prevent surprise breakage)
- Supports **reproducibility** (same CLI version behaves the same everywhere)

---

## 🎛️ HELP & DOCUMENTATION CHECKLIST

### Principle:
The CLI should be self-documenting. Users should understand how to use it without leaving the terminal.

### Checklist

- [ ] **Does `--help` provide complete information for every command?**
  - *Why this is important:* If users can't figure out how to use a command, they won't use it.
  - *news-cli example:* Running `news --help`, `news reddit --help`, `news reddit subscribe --help` all show relevant documentation.

- [ ] **Are examples provided in the help text?**
  - *Why this is important:* "See --help" frustrates users. Show them a quick example.
  - *news-cli example:* Could add examples like: `news reddit subscribe worldnews  # Subscribe to r/worldnews`.

- [ ] **Are error messages helpful (not just "error")?**
  - *Why this is important:* Users should be able to recover from errors without reading external docs.
  - *news-cli example:* "No subreddits subscribed. Use: news reddit subscribe <subreddit>" tells the user what to do.

- [ ] **Is the version accessible via `--version`?**
  - *Why this is important:* Users need to know what version they're running to report bugs accurately.
  - *news-cli example:* `news --version` returns the version from package.json.

- [ ] **Are breaking changes documented?**
  - *Why this is important:* If you remove a command or change behavior, users need to know before they upgrade.
  - *news-cli example:* A CHANGELOG.md or release notes should document version changes.

### How this enforces the principle

- Reduces **support burden** (users figure it out themselves)
- Improves **usability** (intuitive command names and help)
- Enables **discovery** (users learn what's possible)

---

## 🚀 EXIT CODES & SIGNALS CHECKLIST

### Principle:
The CLI communicates success/failure via exit codes. Scripts and automation depend on these signals.

### Checklist

- [ ] **Does the CLI exit with code 0 on success?**
  - *Why this is important:* This is the Unix convention. Scripts check `echo $?` to detect success.
  - *news-cli example:* When `subscribe` completes, the process exits normally with code 0.

- [ ] **Does the CLI exit with a non-zero code on failure?**
  - *Why this is important:* Signals to scripts and automation that something went wrong.
  - *news-cli example:* If no subreddits are subscribed, could exit with code 1 to signal "nothing to do".

- [ ] **Are exit codes documented?**
  - *Why this is important:* Scripts need to know what exit code 42 means.
  - *news-cli example:* Should document: `0 = success`, `1 = validation error`, `2 = network error`, etc.

- [ ] **Does the CLI handle SIGINT (Ctrl+C) gracefully?**
  - *Why this is important:* Allows users to interrupt long-running commands cleanly.
  - *news-cli example:* If `update` is fetching 10 subreddits and the user hits Ctrl+C, the process should exit cleanly without corrupting config.

- [ ] **Are signals (SIGTERM, SIGHUP) handled?**
  - *Why this is important:* In container or systemd environments, processes are terminated via signals. Handlers let you clean up.
  - *news-cli example:* Currently, process.on('SIGTERM') is not set up. Could be added to save state before exiting.

### How this enforces the principle

- Enables **automation** (scripts can detect success/failure)
- Supports **graceful shutdown** (cleanup on interrupt)
- Maintains **data integrity** (state is saved before termination)

---

## 🔐 SECURITY CHECKLIST

### Principle:
CLIs often run with elevated privileges or access sensitive data. Treat security as a first-class concern.

### Checklist

- [ ] **Are API keys ever logged or printed? (If yes → FAIL)**
  - *Why this is important:* API keys in logs or error messages can be stolen.
  - *news-cli example:* Currently, no API keys are stored. If they were added (e.g., for Reddit OAuth), they must never appear in logs.

- [ ] **Is user data (subscriptions) stored securely?**
  - *Why this is important:* Subscriptions file lives in a home directory readable by the user. Acceptable, but consider permissions.
  - *news-cli example:* `~/.news-cli/subscriptions.json` is created with default permissions. Could set mode to 0600 (readable only by the user).

- [ ] **Is the CLI vulnerable to shell injection?**
  - *Why this is important:* If CLI arguments are passed to shell commands without escaping, attackers can inject arbitrary commands.
  - *news-cli example:* Currently, no shell commands are executed. Safe. If subprocess calls are added, use `child_process.execFile()` (not `exec()`).

- [ ] **Are dependencies scanned for vulnerabilities?**
  - *Why this is important:* npm packages can have security flaws. Regular auditing catches them.
  - *news-cli example:* Run `npm audit` before publishing to detect vulnerable dependencies.

- [ ] **Does the CLI validate input to prevent path traversal?**
  - *Why this is important:* If a user can pass `../../../etc/passwd`, they might read arbitrary files.
  - *news-cli example:* Currently, input validation is minimal. If file operations are added, sanitize paths.

### How this enforces the principle

- Prevents **credential leakage**
- Protects **user data**
- Reduces **attack surface**

---

# 🧭 META-CHECK (APPLIES TO ALL PRINCIPLES)

- [ ] What principle is this feature most likely to violate?
- [ ] Where is the weakest boundary in this system?
- [ ] If this fails, what is the blast radius?
- [ ] Is any rule being bypassed for convenience?

**IF YOU CANNOT ANSWER THESE → THE SYSTEM IS NOT UNDERSTOOD**

---

## 🌟 WHY THIS MODULE IS IMPORTANT

This checklist module acts as the **architectural conscience** for CLI tool development. Without these fundamentals, building and reviewing CLI tools becomes guesswork.

Here is why adopting this module is critical for the success of your CLI:

### **Prevents Architectural Drift**
Over time, quick hacks and "temporary" solutions accumulate. A CLI that starts clean can become a tangled mess of hardcoded paths, global state, and tight coupling. These checklists force developers to regularly realign their work with core principles of stable, maintainable CLI tools.

### **Reduces Review Friction**
It removes subjective arguments from code reviews. Instead of debating *opinions* ("should we use chalk or plain colors?"), the team can point to an agreed-upon checklist of *facts* ("is error handling localized?" yes/no).

### **Mitigates Catastrophic Failure**
Most critical failures aren't caused by complex edge cases — they're caused by missing a fundamental principle. A corrupted config file, lost user data, or a crashed process during a write happens because someone overlooked **Process State** or **Configuration Management** principles. This module catches those basic, fatal errors before they reach users.

### **Accelerates Onboarding**
New engineers don't need to guess what the team values or how to structure a CLI. The principles and the "why" behind them are clearly documented. They can write compliant, high-quality code from day one without needing to ask "how do we handle config?" ten times.

### **Enables Scalability**
As the CLI grows (adding new commands, new data sources, more users), a solid foundation of principles prevents technical debt from accumulating. New contributors follow the established patterns and don't introduce entropy.

---

## 🗺️ How to Use This Checklist

**Before implementing a new feature:**
1. Read the relevant checklists (e.g., "External API Boundary" for a new API source).
2. Ask: "What could go wrong here?" and check the items.
3. Design the feature to satisfy all checklist items.

**During code review:**
1. Use the checklists as acceptance criteria, not opinions.
2. If an item is not satisfied, reference it by name ("External API Boundary" → "Are timeouts enforced?").
3. Make it clear: this is not personal preference, it's architectural principle.

**Before publishing:**
1. Run through the Meta-Check section.
2. If you can't answer the questions confidently, the system needs more thought.
3. Don't ship unless you can articulate the principles clearly.

---

## 📚 Reference: news-cli Architecture Summary

```
  ┌──────────────────────────────────────────────┐
  │       news-cli: Complete System              │
  │                                              │
  │  Input Layer:  commander parses argv        │
  │  Logic Layer:  reddit.js, substack.js       │
  │  State Layer:  config.js ↔ subscriptions.json│
  │  Output Layer: chalk, ora, boxen            │
  │                                              │
  │  External APIs: Reddit, Substack            │
  │  External Systems: npm, Node.js, File System│
  └──────────────────────────────────────────────┘
```

Use this as a reference when building or reviewing CLI tools. Every system follows the same patterns at different scales.
