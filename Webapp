# WEB APP SYSTEM PRINCIPLE CHECKLISTS

---

## 🔐 AUTH & IDENTITY CHECKLIST
### Principle:
The system must establish and enforce ownership boundaries.

### Checklist
- [ ] Is identity established before any protected action?
- [ ] Is identity verified at the system boundary (not internally assumed)?
- [ ] Is authorization enforced for every operation involving user data?
- [ ] Is any identity or permission derived from client input? (If yes → FAIL)
- [ ] Can any operation be executed without identity validation? (If yes → FAIL)
- [ ] Are identity checks consistent across all entry points?

### How this enforces the principle
- Prevents **implicit trust leaks** (most common failure)
- Forces identity to be **explicit and validated everywhere**
- Eliminates “weak endpoints” that compromise the system

---

## 🧠 STATE MANAGEMENT CHECKLIST
### Principle:
The system must maintain a consistent and deterministic representation of current state.

### Checklist
- [ ] What is the single source of truth for this state?
- [ ] Is state duplicated across multiple locations? (If yes → justify or FAIL)
- [ ] Can two parts of the system disagree on the same state?
- [ ] Is derived state being stored instead of computed?
- [ ] Are state transitions explicit and traceable?
- [ ] Does the output (UI / behavior) fully derive from state?

### How this enforces the principle
- Eliminates **state divergence**
- Ensures **deterministic behavior**
- Prevents “ghost bugs” caused by unsynced state

---

## 💾 DATA PERSISTENCE CHECKLIST
### Principle:
The system must define what data survives and where truth lives.

### Checklist
- [ ] Does this data need to survive session/device loss?
- [ ] If yes, is it stored in a durable system?
- [ ] Is there exactly ONE source of truth for this data?
- [ ] Is client/local storage being used for critical data? (If yes → FAIL)
- [ ] Can this data be lost without consequence? (If no → must persist safely)
- [ ] Is the persistence strategy aligned with data value?

### How this enforces the principle
- Prevents **silent data loss**
- Forces clarity on **data lifecycle**
- Eliminates multiple conflicting “truths”

---

## 🚧 API BOUNDARY CHECKLIST
### Principle:
All trust decisions must be enforced at the system boundary.

### Checklist
- [ ] Are all external inputs validated at the boundary?
- [ ] Are any critical decisions made outside the boundary? (If yes → FAIL)
- [ ] Does the system assume the caller is trusted? (If yes → FAIL)
- [ ] Are sensitive operations restricted to controlled entry points?
- [ ] Is the boundary clearly defined and consistently enforced?

### How this enforces the principle
- Ensures **zero trust outside the boundary**
- Prevents **client-side bypasses**
- Centralizes security enforcement

---

## 🔑 SECRET MANAGEMENT CHECKLIST
### Principle:
Secrets must never be exposed, persisted insecurely, or misused.

### Checklist
- [ ] Are secrets ever exposed outside trusted environments? (If yes → FAIL)
- [ ] Are secrets stored securely (encrypted or protected)?
- [ ] Are secrets ever logged or included in errors? (If yes → FAIL)
- [ ] Are secrets only accessed at the moment they are needed?
- [ ] Is access to secrets minimized and controlled?

### How this enforces the principle
- Reduces **attack surface**
- Prevents **accidental leaks**
- Ensures secrets exist only when absolutely necessary

---

## ⚡ REAL-TIME COMMUNICATION CHECKLIST
### Principle:
The system must match communication strategy to time-sensitivity.

### Checklist
- [ ] Does the user/system require incremental updates?
- [ ] Is the chosen communication method the simplest that satisfies requirements?
- [ ] Can delays degrade user experience significantly?
- [ ] Is communication unnecessarily complex? (If yes → simplify)
- [ ] Is the system resilient to slow or interrupted communication?

### How this enforces the principle
- Prevents **overengineering (e.g. unnecessary WebSockets)**
- Ensures **UX aligns with system behavior**
- Balances complexity vs responsiveness

---

## 🧱 UI / INTERFACE ARCHITECTURE CHECKLIST
### Principle:
Structure must enable scalability and maintainability.

### Checklist
- [ ] Does each unit/component have a single responsibility?
- [ ] Are responsibilities clearly separated?
- [ ] Are side effects isolated from rendering/logic?
- [ ] Does structure reflect feature boundaries?
- [ ] Are there abstractions with only one use? (If yes → question them)

### How this enforces the principle
- Prevents **tight coupling**
- Ensures **scalability over time**
- Avoids **premature abstraction**

---

## ⚠️ ERROR HANDLING CHECKLIST
### Principle:
Failures must be explicit, categorized, and actionable.

### Checklist
- [ ] Are errors handled at system boundaries?
- [ ] Are different failure types distinguishable?
- [ ] Can users take action based on error messages?
- [ ] Are errors ever silently ignored? (If yes → FAIL)
- [ ] Is debugging information preserved internally?

### How this enforces the principle
- Maintains **observability**
- Enables **debugging**
- Improves **user recovery paths**

---

## 🌐 EXTERNAL INTEGRATION CHECKLIST
### Principle:
External systems must be treated as unreliable.

### Checklist
- [ ] Are timeouts enforced for all external calls?
- [ ] Are failures isolated from the rest of the system?
- [ ] Is retry logic controlled and bounded?
- [ ] Is external data validated before use?
- [ ] Can one external failure cascade into total failure? (If yes → FAIL)

### How this enforces the principle
- Prevents **system-wide crashes**
- Handles **unpredictable failures gracefully**
- Ensures **data integrity**

---

## ☁️ DEPLOYMENT & INFRA CHECKLIST
### Principle:
System design must obey runtime constraints.

### Checklist
- [ ] What are the hard constraints of the environment?
- [ ] Does the system rely on unsupported features?
- [ ] Are there hidden assumptions about execution (e.g. long-running processes)?
- [ ] Will this behave the same in production as in development?
- [ ] Are failure modes in the environment accounted for?

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
