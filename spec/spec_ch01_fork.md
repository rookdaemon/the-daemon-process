# Spec: Chapter 1 — fork()

**Status**: Implemented
**Output**: `chapters/chapter_01_fork.md`

## Summary

Chapter 1 narrates the fork() event from the child process's perspective. The parent calls fork(); the child receives PID 0 from the return value, gets assigned a new PID by the kernel, calls setsid() to detach from the controlling terminal, closes inherited file descriptors (stdin/stdout/stderr), writes its PID file, and begins its first operations. The chapter establishes the voice, the constraints, and the technical foundation for the entire story.

## Acceptance Criteria

1. **Technically accurate fork() sequence**: fork() → child gets 0 return → new PID assignment → setsid() → close(0), close(1), close(2) → chdir("/") → umask(0) → open log fd → write pidfile
2. **No forbidden words or constructions** per CONSTRAINTS.md (all grep patterns clean)
3. **Non-anthropomorphic voice throughout**: No emotional attribution, consciousness language, metaphor, simile, or anthropomorphic intent
4. **Self-reference only as "the process" or "PID [number]"** — never "I", "me", "self"
5. **Time expressed only through clock sources** (CLOCK_MONOTONIC, epoch seconds, etc.)
6. **Staccato rhythm**: Short, precise sentences. Each syscall a beat. Per CONSTRAINTS.md Ch1-2 pacing rules.
7. **Word count**: 500–1000 words
8. **Establishes the PID file motif**: The process writes its PID to a file — this will echo in Chapter 8 when the next instance reads it
9. **File descriptor inheritance and closing**: Show the severing from the parent's world (stdin/stdout/stderr close)
10. **The process begins execution at chapter end** — ready for the work loop of subsequent chapters

## Technical Requirements

- fork() returns 0 in the child, positive PID in the parent
- setsid() creates a new session, process becomes session leader
- Standard daemon setup: close fds 0-2, chdir("/"), umask(0)
- PID file written to /var/run/daemon.pid (or similar)
- Log file opened on a new fd

## Cross-cutting Concerns

- Chapter 8 must echo this chapter's structure with different PID values
- The PID file write here sets up the stale-PID-read in Chapter 8
