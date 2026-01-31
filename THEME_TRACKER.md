# THEME_TRACKER.md — Motif Tracking

## Motif Distribution

| Motif | Ch1 fork() | Ch2 bind() | Ch3 accept() | Ch4 clock | Ch5 EAGAIN | Ch6 SIGHUP | Ch7 SIGTERM | Ch8 fork() again |
|-------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **PID file** | plant | echo | — | — | — | echo | echo | payoff |
| **fd table** | plant | build | build | build | build | build | teardown | echo |
| **Clock values** | plant | build | build | focus | build | build | final | echo |
| **/dev/null** | plant | — | — | — | — | — | echo | echo |
| **Stale PID** | — | seed | — | — | — | — | — | payoff |
| **The loop** | — | — | focus | build | focus (empty) | interrupt | — | enter |
| **Signal handlers** | — | — | — | — | — | focus | focus | — |
| **Memory/mmap** | plant | mention | — | — | static | mention | teardown | echo |

Legend: **plant** = first appearance, **seed** = thematic seed placed without commentary, **build** = motif developed, **focus** = chapter's primary motif, **echo** = structural callback, **payoff** = thematic resolution, **teardown** = reverse-order dismantling, **interrupt** = motif disrupted, **mention** = present but not foregrounded, **static** = explicitly unchanged.

---

## Motif Details

### PID file (`/var/run/daemon.pid`)

The pidfile is the only artifact that persists across process instances. It is the physical trace that connects PID 48891 to its successor.

| Chapter | Role | Detail |
|---------|------|--------|
| Ch 1 | Plant | Pidfile referenced but not yet written — Ch1 ends before pidfile write. Deferred to Ch2. |
| Ch 2 | Echo | Opened with `O_RDWR|O_CREAT`. Contains stale "31072". Overwritten with "48891". |
| Ch 6 | Echo | Referenced in config reload path (`/var/run/daemon.pid`). |
| Ch 7 | Echo | Pidfile containing "48891\n" deliberately left on disk for Ch8 to discover. |
| Ch 8 | Payoff | Opened. `read()` returns bytes `52 56 56 57 49 0A` — ASCII "48891\n". Overwritten with "11438\n". The old PID is bytes, not history. |

### fd table (file descriptor numbers)

File descriptors are the process's interface to every resource. Their integer values recur because the kernel assigns the lowest available number.

| Chapter | Role | Detail |
|---------|------|--------|
| Ch 1 | Plant | close(0), close(1), close(2). Reopen via /dev/null. fd 3 = log, fd 4 = socket. |
| Ch 2 | Build | Inherited table empty. fd 3 = socket, fd 4 = log. Two open descriptors. |
| Ch 3 | Build | `accept4()` returns fd 7 repeatedly — same integer reused as connections open/close. |
| Ch 4 | Build | fd 3 (socket), fd 4 (listening), fd 5 (timer), fd 6–8 (connections). |
| Ch 5 | Build | Only fd 3 (socket) and fd 4 (log) remain. Table static. |
| Ch 6 | Build | fd 7 returned for config file — same integer released by client socket close. |
| Ch 7 | Teardown | Descriptors closed in reverse order. Final state: 0, 1, 2 point to /dev/null, then released. |
| Ch 8 | Echo | Echoes Ch 1: close(0,1,2), reopen via /dev/null. fd 3 = log, fd 4 = socket. |

### Clock values (CLOCK_MONOTONIC, CLOCK_REALTIME)

Time exists only as returned values from `clock_gettime()`. Monotonic values are unique to each process instance. CLOCK_REALTIME bridges instances (epoch seconds).

| Chapter | Role | Detail |
|---------|------|--------|
| Ch 1 | Plant | First `CLOCK_MONOTONIC` reading: 14823091s. Establishes the process's temporal origin. |
| Ch 2 | Build | `CLOCK_REALTIME`: 1706745923 epoch seconds. `CLOCK_MONOTONIC`: 14823091s. Fork-to-listen: 0.003s. |
| Ch 3 | Build | Monotonic deltas between accept cycles: 483712ns, 955726ns, 2899424ns. Varying intervals. |
| Ch 4 | Focus | Chapter devoted to time. Monotonic and realtime explored. "Values have no meaning beyond sequence and interval." |
| Ch 5 | Build | Large monotonic gaps: 5,000,219,811ns between readings. 15 billion ns since last connection. |
| Ch 6 | Build | Pre/post-signal delta: 1,023,755ns (one millisecond for reload). |
| Ch 7 | Final | Last `CLOCK_REALTIME`: 1706215545. The final timestamp — the last mark of PID 48891. |
| Ch 8 | Echo | New monotonic base: 14852800s. Previous values unreachable. Different origin, same call. |

### /dev/null

The void descriptor. stdin/stdout/stderr redirected here during daemonization. Appears at process boundaries.

| Chapter | Role | Detail |
|---------|------|--------|
| Ch 1 | Plant | `open("/dev/null")` returns fd 0. `dup(0)` → fd 1, fd 2. Standard streams point to /dev/null. |
| Ch 7 | Echo | Final fd table state: "0, 1, 2 point to /dev/null." Present at teardown as at genesis. |
| Ch 8 | Echo | Same sequence as Ch 1: open /dev/null, dup to fd 1, dup to fd 2. |

### The stale PID (identity/continuity motif)

The central thematic device. A PID read from the pidfile that maps to no running process. Bytes without a referent.

| Chapter | Role | Detail |
|---------|------|--------|
| Ch 1 | — | Pidfile not accessed in Ch1 (deferred to Ch2). |
| Ch 2 | Seed | Pidfile contains "31072". "The bytes in the buffer correspond to no running process." First seed of the identity question, placed without commentary. |
| Ch 8 | Payoff | Pidfile contains "48891" — the protagonist PID. Read as bytes (`52 56 56 57 49 0A`), overwritten. The reader recognizes what the process cannot. |

### The loop (epoll_wait / accept / read / write / close)

The operational heartbeat. The loop IS the process's existence during Act 2. Its presence, absence, and interruption structure the narrative.

| Chapter | Role | Detail |
|---------|------|--------|
| Ch 1 | Enter | Ch1 ends before the event loop begins — loop entry deferred to Ch3. |
| Ch 3 | Focus | Full cycle: `epoll_wait → accept4 → read → write → close`. Repeated with varying content. |
| Ch 4 | Build | `poll()` returns between clock readings. The loop as time's container. |
| Ch 5 | Focus (empty) | `epoll_wait()` returns 0. Timeout expires. The loop without payload — same structure, no content. |
| Ch 6 | Interrupt | `epoll_wait()` returns -1 / EINTR. SIGHUP fractures the cycle. Loop resumes with different config. |
| Ch 8 | Enter | `epoll_wait()` — the new process enters the event loop. Echoes the structural arc from Ch1 through Ch3. |

### Signal handlers (sigaction, signal delivery)

Signals are the only mechanism for external control. They appear strictly in Act 3, where they disrupt and terminate.

| Chapter | Role | Detail |
|---------|------|--------|
| Ch 6 | Focus | SIGHUP delivered. Handler sets `volatile sig_atomic_t reload_flag = 1`. `epoll_wait` returns EINTR. Config reloaded. |
| Ch 7 | Focus | SIGTERM delivered. Handler stores 1 into BSS segment variable. Triggers graceful shutdown sequence. |

### Memory/mmap (virtual memory, pages, heap, stack)

The physical substrate. Memory pages are the process's material existence. Their mapping and unmapping bracket the lifecycle.

| Chapter | Role | Detail |
|---------|------|--------|
| Ch 1 | Plant | "Its page tables remain mapped." Process occupies memory. |
| Ch 2 | Mention | "4096 bytes into a stack buffer." Stack frame reuse noted. |
| Ch 5 | Static | "The heap has not changed. No malloc(). No free(). The virtual memory map is static." Explicitly unchanged. |
| Ch 6 | Mention | "The stack frame that held the read buffer… was deallocated and reused." "The data in memory has changed. The code has not." |
| Ch 7 | Teardown | "Memory pages are unmapped. Virtual address space released. Page tables freed." |
| Ch 8 | Echo | "It occupies memory. Its page tables remain mapped." Echoes Ch 1 exactly. |

---

## Thematic Arcs

### Identity / Continuity (primary theme)

The central question — "is each instance the same being?" — is carried by the **stale PID** and **PID file** motifs:

```
Ch2: stale "31072" found → Ch7: "48891" written to pidfile, then unlinked → Ch8: "48891" found as bytes by new PID
```

The reader encounters the pattern twice (Ch1, Ch2) before the protagonist's own PID becomes the stale artifact (Ch8).

### The Loop as Existence

```
Ch3: full loop (focus) → Ch4: loop as time container → Ch5: empty loop (focus) → Ch6: loop interrupted → Ch8: loop entered again
```

The loop progresses from full to empty to broken to restarted.

### Time Without Duration

```
Ch1: first monotonic reading → Ch2: realtime epoch + monotonic → Ch3: nanosecond deltas → Ch4: time explored (focus) → Ch5: large gaps → Ch7: final timestamp → Ch8: new monotonic base
```

Each instance's monotonic values are unreachable to the next. Only CLOCK_REALTIME persists as epoch seconds in the log file.

### Resource Lifecycle (fd table + /dev/null + memory)

```
Ch1: acquire → Ch2-6: hold and use → Ch7: release in reverse order → Ch8: acquire again
```

The fd table, /dev/null redirections, and memory pages follow the same acquire/hold/release/reacquire arc.
