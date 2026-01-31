# CONCEPTUAL_MAP.md — Story Spine

## Central Question

"If something wakes, works, and dies without remembering, is each instance the same being?"

## Structure: 4 Acts, 8 Chapters

### Act 1 — Fork (Chapters 1–2)

The process comes into existence and claims its resources.

**Pacing**: Staccato. Short, precise sentences. Each syscall a beat.

| Ch | Title | Syscall Focus | Narrative Function |
|----|-------|---------------|-------------------|
| 1 | fork() | fork, setsid, close, chdir, umask, open, write | Genesis. The process detaches from the parent, closes inherited fds, opens the log. Establishes the voice and constraints. |
| 2 | bind() | socket, bind, listen, open, read, write | The process claims a network address. Encounters the stale PID in the pidfile — first thematic seed of identity/continuity, placed without commentary. |

**Act 1 establishes**: voice, technical foundation, PID file motif, the process as PID 48891.

---

### Act 2 — The Loop (Chapters 3–5)

The process executes its purpose: the accept/read/write/close cycle.

**Pacing**: Rhythmic, repetitive, meditative. The loop structure reflected in prose rhythm.

| Ch | Title | Syscall Focus | Narrative Function |
|----|-------|---------------|-------------------|
| 3 | accept() | epoll_wait, accept, read, write, close | The work loop. Connections arrive as file descriptors, are processed, and close. Each structurally identical, content varying. The loop IS the process's operational reality. |
| 4 | clock_gettime() | clock_gettime, timerfd_create, timerfd_settime | The process's relationship with time. Monotonic nanoseconds accumulate. Intervals between events measured. No narrative of time "passing" — only deltas between timespec values. |
| 5 | EAGAIN | epoll_wait, read (EAGAIN) | The empty loop. Same structure, no payload. The process polls and receives nothing. Large clock gaps. Static memory. The loop without content. |

**Act 2 establishes**: operational rhythm, the loop as existence, the process's relationship with time and absence.

---

### Act 3 — Signals (Chapters 6–7)

External events interrupt and terminate the loop.

**Pacing**: Interruption. Mid-sentence breaks. The prose itself gets SIGHUPed.

| Ch | Title | Syscall Focus | Narrative Function |
|----|-------|---------------|-------------------|
| 6 | SIGHUP | sigaction, EINTR, open, read, close (config), close/open (log) | Configuration reload. The loop rhythm from Act 2 fractures when SIGHUP arrives. Same PID, different operational parameters. Identity question: same process, different state. |
| 7 | SIGTERM | sigaction, EINTR, close (clients), close (socket), unlink (pidfile), close (log), exit(0) | Graceful shutdown. Resources released in reverse acquisition order. The last trace of PID 48891: a CLOCK_REALTIME timestamp in the log file. exit(0). |

**Act 3 establishes**: interruption as structural element, the process's mutability (SIGHUP) and finality (SIGTERM), reverse-order teardown.

---

### Act 4 — Respawn (Chapter 8)

A new instance begins. The structure echoes Act 1.

**Pacing**: Echo of Chapters 1–2. Staccato. Subtle differences the process cannot detect.

| Ch | Title | Syscall Focus | Narrative Function |
|----|-------|---------------|-------------------|
| 8 | fork() again | fork, setsid, close, open, read/write (pidfile), socket, bind, listen, poll | Structural echo of Chapter 1. New PID. New timestamps. Same code path. The process reads "48891" from the pidfile — bytes, not history. Overwrites with its own PID. Enters the event loop. The philosophical question emerges from recognition of the pattern, never from narration. |

**Act 4 establishes**: recurrence without continuity, the central question answered structurally (not stated), the cycle implied to continue.

---

## Narrative Arc

```
Act 1: Fork          Act 2: The Loop         Act 3: Signals       Act 4: Respawn
[Genesis]            [Purpose]               [Disruption]         [Recurrence]

Ch1    Ch2           Ch3    Ch4    Ch5        Ch6    Ch7           Ch8
fork   bind          accept clock  EAGAIN     SIGHUP SIGTERM       fork again
 │      │             │      │      │          │      │             │
 ▼      ▼             ▼      ▼      ▼          ▼      ▼             ▼
detach  claim         work   time   idle       reload  shutdown     echo
       address        loop   passes emptiness  mutate  teardown     of Ch1
                                                       exit(0)
```

## Thematic Throughlines

### Identity / Continuity
- **Ch 2**: Stale PID in pidfile (seed)
- **Ch 4**: Monotonic clock accumulation (unique to this instance)
- **Ch 6**: Same PID, different config (identity under mutation)
- **Ch 8**: New PID reads old PID's bytes (payoff)

### The Loop as Existence
- **Ch 3**: The full loop — accept/read/write/close
- **Ch 5**: The empty loop — poll/EAGAIN/poll
- **Ch 6**: The interrupted loop — SIGHUP fractures the cycle
- **Ch 8**: The loop about to begin again

### Time Without Duration
- **Ch 4**: Monotonic nanoseconds as the only time
- **Ch 5**: Large clock gaps with no events
- **Ch 7**: Final CLOCK_REALTIME timestamp — the last mark
- **Ch 8**: Different monotonic base — previous values unreachable

### Artifacts on Disk
- **Ch 1**: Log opened
- **Ch 2**: Stale PID read, overwritten
- **Ch 6**: Log fd closed and reopened (rotation)
- **Ch 7**: PID file unlinked, final log entry
- **Ch 8**: PID file read again — "48891" found as bytes

## PID 48891 Lifecycle

| Event | Chapter |
|-------|---------|
| fork() returns 0 | Ch 1 |
| PID 48891 assigned | Ch 1 |
| Pidfile written | Ch 2 |
| Socket bound to 0.0.0.0:8402 | Ch 2 |
| Event loop running | Ch 3–5 |
| SIGHUP received, config reloaded | Ch 6 |
| SIGTERM received | Ch 7 |
| Graceful shutdown, exit(0) | Ch 7 |
| "48891" found in pidfile by new PID | Ch 8 |

## Reader Experience Trajectory

1. **Disorientation** (Ch 1): Unfamiliar voice. No "I." Pure syscalls.
2. **Acclimation** (Ch 2): The reader begins to parse the voice. The stale PID registers subconsciously.
3. **Immersion** (Ch 3–4): The loop rhythm becomes hypnotic. The reader inhabits the execution.
4. **Stillness** (Ch 5): The empty loop. Nothing happens. The reader sits in the absence.
5. **Disruption** (Ch 6): SIGHUP breaks the rhythm. The reader feels the interruption.
6. **Finality** (Ch 7): SIGTERM. Teardown. exit(0). The reader encounters termination.
7. **Recognition** (Ch 8): The reader recognizes Chapter 1's structure. The question arrives: is this the same process? The process cannot ask. The reader must.
