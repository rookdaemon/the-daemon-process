# Spec: Chapter 7 — SIGTERM

**Status**: Implemented
**Output**: `chapters/chapter_07_sigterm.md`

## Summary

Chapter 7 is the graceful shutdown. SIGTERM is delivered to the process. The signal handler sets a flag. The event loop checks the flag. The process begins orderly teardown: closing client connections, draining buffers, closing the listening socket, writing a final log entry, removing the pidfile, closing the log file descriptor. Each resource opened in Chapters 1-4 is released in reverse order. The prose rhythm is interrupted — mid-operation breaks, the signal arriving between syscalls. The chapter ends with exit(0).

## Acceptance Criteria

1. **SIGTERM delivery is the central event**: Signal arrives via kill(pid, SIGTERM) from an external process (supervisor, init, or admin). The signal handler executes, setting a volatile sig_atomic_t flag.
2. **No forbidden words or constructions** per CONSTRAINTS.md (all grep patterns clean)
3. **Non-anthropomorphic voice throughout**: No emotional attribution, consciousness language, metaphor, simile, or anthropomorphic intent
4. **Self-reference only as "the process" or "PID 48891"** — never "I", "me", "self"
5. **Interrupted rhythm**: Per CONSTRAINTS.md Ch6-7 pacing — mid-sentence breaks, the prose disrupted by signal delivery. The signal interrupts the event loop.
6. **Graceful shutdown sequence**: The process closes client file descriptors, closes the listening socket, writes a final log entry, removes the pidfile, closes the log fd, and calls exit(0)
7. **Reverse-order teardown**: Resources acquired in Chapters 1-4 are released approximately in reverse order (client fds first, then listening socket, then pidfile, then log)
8. **Word count**: 500-1000 words
9. **Continuity with prior chapters**: Same PID 48891, same fd numbers (3=log, 4=listening socket, 5=epoll, 6=timerfd), same monotonic clock range continuing from Chapter 4
10. **Signal handler mechanics shown accurately**: sigaction() or signal() established disposition, handler runs asynchronously, sets flag, returns. Main loop checks flag.
11. **Final log entry**: The process writes a shutdown log line with CLOCK_REALTIME timestamp before closing fd 3
12. **pidfile removal**: unlink("/var/run/daemon.pid") before exit
13. **exit(0)**: The chapter ends with exit(0). The process terminates with status 0.

## Technical Requirements

- SIGTERM (signal 15) delivered by kill() from another process
- Signal handler: void handler(int sig) { shutdown_flag = 1; }
- volatile sig_atomic_t for the flag (async-signal-safe)
- close() for each open client fd
- close(4) for listening socket
- unlink() for pidfile
- write() final log entry to fd 3
- close(3) for log
- exit(0)

## Cross-cutting Concerns

- Chapter 8 (Respawn) will show a new process starting — the supervisor detects exit and forks again. The pidfile will be gone (unlinked here). The new process will write a new pidfile.
- The monotonic clock values from this chapter will never be referenced again — the new process gets a different monotonic base.
- The CLOCK_REALTIME timestamp in the final log entry will be the last trace of PID 48891 on disk.
