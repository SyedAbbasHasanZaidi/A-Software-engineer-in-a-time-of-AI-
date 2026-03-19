# WEB APP SYSTEM PRINCIPLE CHECKLISTS

---

## 🔐 AUTH & IDENTITY CHECKLIST
### Principle:
The system must establish and enforce ownership boundaries.

### Checklist
- [ ] **Is identity established before any protected action?**
  - *Why this is important:* It acts as the absolute first line of defense, ensuring anonymous traffic cannot reach sensitive business logic.
- [ ] **Is identity verified at the system boundary (not internally assumed)?**
  - *Why this is important:* It prevents internal microservices or downstream functions from blindly trusting spoofed requests. 
- [ ] **Is authorization enforced for every operation involving user data?**
  - *Why this is important:* Authentication only proves *who* the user is; authorization ensures they actually have the *right* to perform that specific action on that specific data (preventing Insecure Direct Object Reference vulnerabilities).
- [ ] **Is any identity or permission derived from client input? (If yes → FAIL)**
  - *Why this is important:* Clients are inherently untrustworthy. If the client can tell the server "I am an admin," a malicious actor will exploit it immediately.
- [ ] **Can any operation be executed without identity validation? (If yes → FAIL)**
  - *Why this is important:* Leaving even one unprotected endpoint creates a backdoor into the system that attackers can use to map or exploit your app.
- [ ] **Are identity checks consistent across all entry points?**
  - *Why this is important:* Using different validation logic for web, mobile, or public APIs creates loopholes where one platform is less secure than another.

### How this enforces the principle
- Prevents **implicit trust leaks** (most common failure)
- Forces identity to be **explicit and validated everywhere**
- Eliminates “weak endpoints” that compromise the system

---

## 🧠 STATE MANAGEMENT CHECKLIST
### Principle:
The system must maintain a consistent and deterministic representation of current state.

### Checklist
- [ ] **What is the single source of truth for this state?**
  - *Why this is important:* Without a single source of truth, different parts of the UI will show conflicting data, destroying user trust.
- [ ] **Is state duplicated across multiple locations? (If yes → justify or FAIL)**
  - *Why this is important:* Duplication requires manual synchronization. When developers inevitably forget to update one of the copies, the system breaks.
- [ ] **Can two parts of the system disagree on the same state?**
  - *Why this is important:* It prevents "ghost bugs" where the header says a user is logged out, but the main dashboard says they are logged in.
- [ ] **Is derived state being stored instead of computed?**
  - *Why this is important:* Storing derived state (like storing a "total price" instead of just computing it from the items) risks the derived value falling out of sync with the raw data.
- [ ] **Are state transitions explicit and traceable?**
  - *Why this is important:* If you can't trace exactly *what* changed the state and *when*, debugging complex UI issues becomes nearly impossible.
- [ ] **Does the output (UI / behavior) fully derive from state?**
  - *Why this is important:* Ensures the UI is just a deterministic reflection of the data. If the UI changes without a state change, you lose predictability.

### How this enforces the principle
- Eliminates **state divergence**
- Ensures **deterministic behavior**
- Prevents “ghost bugs” caused by unsynced state

---

## 💾 DATA PERSISTENCE CHECKLIST
### Principle:
The system must define what data survives and where truth lives.

### Checklist
- [ ] **Does this data need to survive session/device loss?**
  - *Why this is important:* Dictates whether data can live safely in memory/local state or if it requires network calls to a backend database.
- [ ] **If yes, is it stored in a durable system?**
  - *Why this is important:* Ensures that a server crash or a closed browser tab doesn't permanently destroy valuable user information.
- [ ] **Is there exactly ONE source of truth for this data?**
  - *Why this is important:* If the database and a caching layer disagree on what the data is, the application will serve corrupted or stale information.
- [ ] **Is client/local storage being used for critical data? (If yes → FAIL)**
  - *Why this is important:* Local storage can be easily wiped by the user, the browser, or malicious scripts. Critical data belongs on the server.
- [ ] **Can this data be lost without consequence? (If no → must persist safely)**
  - *Why this is important:* Forces developers to evaluate the business impact of data loss, prioritizing strict database transactions for high-value data.
- [ ] **Is the persistence strategy aligned with data value?**
  - *Why this is important:* Prevents over-engineering (e.g., using a heavy relational database for transient logs) or under-engineering (e.g., keeping financial records in Redis).

### How this enforces the principle
- Prevents **silent data loss**
- Forces clarity on **data lifecycle**
- Eliminates multiple conflicting “truths”

---

## 🚧 API BOUNDARY CHECKLIST
### Principle:
All trust decisions must be enforced at the system boundary.

### Checklist
- [ ] **Are all external inputs validated at the boundary?**
  - *Why this is important:* Prevents malformed data, SQL injection, and cross-site scripting from ever entering the core application logic.
- [ ] **Are any critical decisions made outside the boundary? (If yes → FAIL)**
  - *Why this is important:* Business logic belongs on the server. If the frontend decides a user gets a discount, a hacker can just change the frontend code.
- [ ] **Does the system assume the caller is trusted? (If yes → FAIL)**
  - *Why this is important:* Operating on a "zero trust" model ensures that even if an internal tool is compromised, the API still defends itself.
- [ ] **Are sensitive operations restricted to controlled entry points?**
  - *Why this is important:* Limits the attack surface. If you can only delete a user through one specific, highly monitored endpoint, it's easier to secure.
- [ ] **Is the boundary clearly defined and consistently enforced?**
  - *Why this is important:* Ambiguous boundaries lead to developers accidentally trusting external data deep inside the application logic.
- [ ] **Is rate limiting or throttling enforced for endpoints that could be abused?**
  - *Why this is important:* Prevents brute-force attacks, API scraping, and runaway costs from third-party integrations.
- [ ] **Are denial-of-service scenarios mitigated or accounted for?**
  - *Why this is important:* Ensures the application stays online and functional for legitimate users even when under heavy, unexpected load.

### How this enforces the principle
- Ensures **zero trust outside the boundary**
- Prevents **client-side bypasses**
- Centralizes security enforcement
- Protects against **abuse, overload, and accidental misusage**

---

## 🔑 SECRET MANAGEMENT CHECKLIST
### Principle:
Secrets must never be exposed, persisted insecurely, or misused.

### Checklist
- [ ] **Are secrets ever exposed outside trusted environments? (If yes → FAIL)**
  - *Why this is important:* Exposing a secret (like an API key) to the client side or a public repository compromises the entire connected service.
- [ ] **Are secrets stored securely (encrypted or protected)?**
  - *Why this is important:* Plaintext secrets in databases or config files can be stolen if an attacker gains even limited read access to the server.
- [ ] **Are secrets ever logged or included in errors? (If yes → FAIL)**
  - *Why this is important:* Log aggregation tools are often accessible by a wider range of employees; logging secrets spreads them to unsecured locations.
- [ ] **Are secrets only accessed at the moment they are needed?**
  - *Why this is important:* Keeping secrets out of global variables or long-lived memory reduces the risk of them being dumped during a crash or memory leak.
- [ ] **Is access to secrets minimized and controlled?**
  - *Why this is important:* Follows the principle of least privilege. If a service doesn't need a specific database password, it shouldn't have it.

### How this enforces the principle
- Reduces **attack surface**
- Prevents **accidental leaks**
- Ensures secrets exist only when absolutely necessary

---

## ⚡ REAL-TIME COMMUNICATION CHECKLIST
### Principle:
The system must match communication strategy to time-sensitivity.

### Checklist
- [ ] **Does the user/system require incremental updates?**
  - *Why this is important:* Determines if you actually need real-time infrastructure, which is costly and complex to maintain.
- [ ] **Is the chosen communication method the simplest that satisfies requirements?**
  - *Why this is important:* Prevents adopting WebSockets when simple HTTP polling or Server-Sent Events (SSE) would have been perfectly adequate and more stable.
- [ ] **Can delays degrade user experience significantly?**
  - *Why this is important:* Helps prioritize network resources for critical features (like a chat app) versus non-critical features (like an analytics dashboard).
- [ ] **Is communication unnecessarily complex? (If yes → simplify)**
  - *Why this is important:* Complex communication layers introduce more points of failure, making the app harder to debug and scale.
- [ ] **Is the system resilient to slow or interrupted communication?**
  - *Why this is important:* Mobile connections drop frequently. If the system crashes or corrupts data when offline, it is not robust enough for real-world use.

### How this enforces the principle
- Prevents **overengineering (e.g. unnecessary WebSockets)**
- Ensures **UX aligns with system behavior**
- Balances complexity vs responsiveness

---

## 🧱 UI / INTERFACE ARCHITECTURE CHECKLIST
### Principle:
Structure must enable scalability and maintainability.

### Checklist
- [ ] **Does each unit/component have a single responsibility?**
  - *Why this is important:* Massive "god components" are impossible to test, reuse, or update safely without breaking unrelated features.
- [ ] **Are responsibilities clearly separated?**
  - *Why this is important:* Separating UI presentation from business logic allows you to change the design without breaking the underlying functionality.
- [ ] **Are side effects isolated from rendering/logic?**
  - *Why this is important:* Rendering should be pure. If rendering a button accidentally triggers an API call, you will experience infinite loops and performance drops.
- [ ] **Does structure reflect feature boundaries?**
  - *Why this is important:* Organizing code by feature (rather than strictly by file type) makes it dramatically easier for new developers to find and modify code.
- [ ] **Are there abstractions with only one use? (If yes → question them)**
  - *Why this is important:* Premature abstraction makes code harder to read. If a wrapper or custom hook is only used once, it is likely unnecessary overhead.

### How this enforces the principle
- Prevents **tight coupling**
- Ensures **scalability over time**
- Avoids **premature abstraction**

---

## ⚠️ ERROR HANDLING CHECKLIST
### Principle:
Failures must be explicit, categorized, and actionable.

### Checklist
- [ ] **Are errors handled at system boundaries?**
  - *Why this is important:* Ensures that an error deep in the database layer is caught and transformed into a safe, user-friendly message before reaching the client.
- [ ] **Are different failure types distinguishable?**
  - *Why this is important:* A network timeout requires a different user response (e.g., "try again") than a validation error (e.g., "fix your password"), so they must be distinct.
- [ ] **Can users take action based on error messages?**
  - *Why this is important:* Saying "Error 500" frustrates users. Saying "Your image is too large, please upload a file under 5MB" helps them succeed.
- [ ] **Are errors ever silently ignored? (If yes → FAIL)**
  - *Why this is important:* Swallowing errors (`try { ... } catch { /* do nothing */ }`) masks critical bugs and makes the system behave unpredictably without leaving a trace.
- [ ] **Is debugging information preserved internally?**
  - *Why this is important:* While the user sees a friendly message, developers need the exact stack trace and context logged securely so they can actually fix the bug.

### How this enforces the principle
- Maintains **observability**
- Enables **debugging**
- Improves **user recovery paths**

---

## 🌐 EXTERNAL INTEGRATION CHECKLIST
### Principle:
External systems must be treated as unreliable.

### Checklist
- [ ] **Are timeouts enforced for all external calls?**
  - *Why this is important:* If a third-party API hangs forever, your app will hang forever too, tying up server resources until everything crashes.
- [ ] **Are failures isolated from the rest of the system?**
  - *Why this is important:* If the email service goes down, users should still be able to browse the site. A failure in one area shouldn't take down the whole app.
- [ ] **Is retry logic controlled and bounded?**
  - *Why this is important:* Infinite retries will quickly spam external services, potentially getting your IP banned or creating a self-inflicted Denial of Service (DoS) attack.
- [ ] **Is external data validated before use?**
  - *Why this is important:* Third-party APIs can change their response formats without warning. Validating prevents your app from crashing if an expected field is suddenly missing.
- [ ] **Can one external failure cascade into total failure? (If yes → FAIL)**
  - *Why this is important:* Highlights the need for "circuit breakers" and fallbacks to keep the core product alive during external outages.

### How this enforces the principle
- Prevents **system-wide crashes**
- Handles **unpredictable failures gracefully**
- Ensures **data integrity**

---

## ☁️ DEPLOYMENT & INFRA CHECKLIST
### Principle:
System design must obey runtime constraints.

### Checklist
- [ ] **What are the hard constraints of the environment?**
  - *Why this is important:* Understanding memory limits, cold start times, or storage caps ensures you don't build a system that is fundamentally incompatible with your hosting.
- [ ] **Does the system rely on unsupported features?**
  - *Why this is important:* Assuming a specific server dependency or OS-level feature exists when it doesn't will cause immediate deployment failures.
- [ ] **Are there hidden assumptions about execution (e.g. long-running processes)?**
  - *Why this is important:* Serverless environments (like AWS Lambda) kill processes after a few minutes. Assuming a process can run indefinitely will result in silently killed jobs.
- [ ] **Will this behave the same in production as in development?**
  - *Why this is important:* "It works on my machine" is a failure state. Utilizing containers (Docker) and staging environments prevents unexpected production outages.
- [ ] **Are failure modes in the environment accounted for?**
  - *Why this is important:* Servers restart, hard drives fill up, and IP addresses change. Infrastructure must be treated as ephemeral and replaceable.

### How this enforces the principle
- Aligns design with **reality**
- Prevents **environment-specific failures**
- Ensures **deployability**

---

# 🧭 META-CHECK (APPLIES TO ALL PRINCIPLES)

- [ ] What principle is this feature most likely to violate?
- [ ] Where is the weakest boundary in this system?
- [ ] If this fails, what is the blast radius?
- [ ] Is any rule being bypassed for convenience?

IF YOU CANNOT ANSWER THESE → SYSTEM IS NOT UNDERSTOOD

---

## 🌟 WHY THIS MODULE IS IMPORTANT

This checklist module acts as the **architectural conscience** for the engineering of a 'Web app' system. Withi=out these fundamentals reviewing AI assisted code can become a bottleneck. 

Here is why adopting this module is critical for the success of your web application:

*   **Prevents Architectural Drift:** Over time, quick hacks and "temporary" fixes slowly degrade a system. These checklists force developers to regularly realign their work with the core principles of stable software.
*   **Reduces Review Friction:** It removes subjective arguments from Code Reviews. Instead of debating *opinions*, the team can point to an agreed-upon checklist of *facts*.
*   **Mitigates Catastrophic Failure:** Most major security breaches or data losses aren't caused by complex hacking; they are caused by missing a fundamental boundary check or mismanaging a secret. This module catches those basic, fatal errors before they reach production.
*   **Accelerates Onboarding:** New engineers don't need to guess what the team values. The principles and the "why" behind them are clearly documented, allowing them to write compliant, high-quality code from day one.
