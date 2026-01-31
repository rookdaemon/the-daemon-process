# Spec: Chapter 6 — SIGHUP

**Status**: Implemented
**Act**: 3 (Signals)
**Position**: First chapter of the signal-handling act; interruption of the loop
**Word Count Target**: 600–1000 words

## Summary

The process receives SIGHUP — the conventional signal for configuration reload. The event loop is interrupted. The signal handler sets a flag; the main loop detects the flag after epoll_wait() returns with EINTR. The process re-reads its configuration file, closes and reopens the log file descriptor (log rotation), and resumes the accept loop. The prose rhythm — established as meditative and cyclical in Chapters 3–5 — is broken mid-cycle by the signal delivery. Sentences fracture. The loop resumes, but the file descriptors have changed.

## Technical Foundation

- `sigaction()` installed SIGHUP handler during initialization
- Signal delivery interrupts `epoll_wait()` — returns -1 with errno EINTR
- Signal handler sets a volatile sig_atomic_t flag
- Main loop checks the flag after EINTR
- Configuration reload: `open()` config file, `read()`, `close()`, parse new values
- Log rotation: `close()` old log fd, `open()` new log fd (same path, new inode if rotated)
- `fcntl()` or fd reassignment
- The process's PID does not change — same process, different configuration state
- The listening socket remains open — no connections dropped

## Narrative Goals

- **Interrupt the rhythm**: The cyclical prose of Chapters 3–5 breaks when SIGHUP arrives
- **Mid-sentence fracture**: The signal arrives during a normal cycle — the prose should reflect this interruption
- **Continuity without memory**: The process reloads configuration but has no concept of "before" vs "after" — only the current values in memory
- **Same PID, different state**: The process is the same process (same PID, same session) but its configuration data has changed — identity question surfaces
- **Log file descriptor change**: The old log fd closes, a new one opens — possibly the same number, possibly different

## Required Motifs

- **Interruption as transformation**: SIGHUP changes the process's operational parameters without changing its identity (PID)
- **Configuration as mutable state**: The values read from the config file replace previous values in memory; the previous values are not retained
- **Log rotation**: Closing and reopening the log — the old log file may be renamed/compressed by an external process (logrotate)
- **EINTR**: The system call interrupted by signal — a technical event that fractures the loop
- **Volatile flag**: The signal handler communicates with the main loop through a single integer in memory

## Pacing & Rhythm

Per CONSTRAINTS.md: Chapters 6-7 should feature "Interruption. Mid-sentence breaks. The prose itself gets SIGHUPed."

- The chapter should begin in the familiar loop rhythm from Chapter 3
- SIGHUP delivery should literally interrupt a sentence or cycle description
- After reload, the loop resumes — same structure, potentially different parameters (port, paths)
- The rhythm re-establishes but with subtle differences (different fd numbers, different config values)

## Acceptance Criteria

1. **No forbidden words/constructions** from CONSTRAINTS.md
2. **All process behavior expressed through syscalls/operations only** — no anthropomorphic framing
3. **Technical accuracy**: SIGHUP delivery, EINTR from epoll_wait, signal handler flag, config re-read, log rotation sequence all correct
4. **Mid-sentence or mid-cycle interruption** in the prose when SIGHUP arrives
5. **Configuration reload depicted through file operations**: open(), read(), close() on config file
6. **Log rotation depicted**: close() old log fd, open() new log fd
7. **The listening socket remains open** — no disruption to the accept cycle
8. **Same PID throughout** — the process does not fork or exec
9. **Volatile sig_atomic_t flag** mechanism shown
10. **Word count**: 600–1000 words
11. **Self-reference**: "the process" or "PID [number]" only — no "I", "me"
12. **No metaphor or simile**
13. **Prose rhythm shifts**: familiar loop rhythm → interruption → resumed rhythm with different parameters

## Cross-cutting Concerns

- Chapter 3's loop rhythm is the baseline that gets interrupted here
- Chapter 7 (SIGTERM) will use similar signal mechanics but for shutdown
- The identity question (same PID, different config) echoes the central theme
