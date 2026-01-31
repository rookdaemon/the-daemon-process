# Spec: Chapter 2 — bind()

**Status**: Implemented
**Chapter**: 2 of 8
**Act**: Act 1 (Fork)
**Working Title**: bind()
**Output File**: `chapters/chapter_02_bind.md`

---

## Summary

Chapter 2 covers the daemon's transition from a newly-forked, detached process to one that claims a network address. After setsid() and fd cleanup in Chapter 1, the process now calls socket(), bind(), and listen(). It reads its configuration file. It writes its PID file — possibly finding a stale PID from a previous instance. It opens its log file descriptor. By chapter end, the process is bound to its address and ready to accept connections.

This chapter introduces the first encounter with artifacts left by a previous process instance — the stale PID file — which seeds the philosophical question of identity/continuity without the process "noticing" anything.

---

## Narrative Beats

1. **socket()** — The process creates a socket file descriptor (AF_INET, SOCK_STREAM).
2. **Configuration read** — open(), read() on the config file. Parse port number, address to bind.
3. **bind()** — The process binds the socket to an address:port. On EADDRINUSE, the process may find SO_REUSEADDR necessary.
4. **PID file encounter** — open() on the PID file path. The file exists. read() returns bytes: a different PID number. The process writes its own PID, overwriting. No commentary — just the syscalls.
5. **listen()** — The backlog queue is set. The socket is now passive.
6. **Log file open** — open() with O_APPEND | O_CREAT. First write(): a timestamp and "listening" status.

---

## Technical Requirements

- Accurate socket(AF_INET, SOCK_STREAM, 0) → fd semantics
- Accurate bind() with struct sockaddr_in
- SO_REUSEADDR setsockopt before bind
- PID file open/read/write sequence (open with O_RDWR | O_CREAT, read existing content, lseek, truncate, write new PID)
- listen() with backlog parameter
- Log file open with O_WRONLY | O_APPEND | O_CREAT and mode bits
- All time references via clock_gettime(CLOCK_MONOTONIC) or epoch seconds

---

## Acceptance Criteria

1. **AC-1**: Chapter contains accurate socket(), bind(), listen() sequence with correct argument semantics.
2. **AC-2**: PID file encounter is present — process reads stale PID, overwrites with own PID. No emotional or consciousness language surrounds this moment.
3. **AC-3**: Configuration file is read via open()/read() syscalls (not abstracted).
4. **AC-4**: No forbidden words or constructions from CONSTRAINTS.md are present (verified by grep).
5. **AC-5**: No metaphors, similes, or anthropomorphic intent language.
6. **AC-6**: Self-reference uses only "the process" or "PID [number]" — never "I", "me", "self".
7. **AC-7**: Time expressed only through clock sources (monotonic nanoseconds, epoch seconds), never human temporal concepts.
8. **AC-8**: Prose rhythm is short, precise, staccato — matching Act 1 pacing (CONSTRAINTS.md Chapters 1-2).
9. **AC-9**: Word count between 500–1000 words.
10. **AC-10**: Chapter ends with the process in a listening state, ready for accept() (Chapter 3 handoff).
11. **AC-11**: The stale PID in the pidfile seeds the identity/continuity theme without explicit commentary.

---

## Cross-Cutting Concerns

- Must be consistent with Chapter 1's ending state (process has PID, session leader via setsid(), stdin/stdout/stderr closed)
- Must set up state that Chapter 3 (accept()) expects: a listening socket fd
- PID file encounter is the first thematic seed — handle with restraint

---

## Motifs to Place

- **Stale PID / overwrite**: Identity without continuity
- **bind()**: Claiming a place in the address space — existence made visible to the network
- **First log write**: The process's first mark on the filesystem
