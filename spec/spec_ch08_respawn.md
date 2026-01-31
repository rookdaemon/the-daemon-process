# Spec: Chapter 8 — fork() again / Respawn

**Status**: Implemented
**Output**: `chapters/chapter_08_respawn.md`

## Summary

Chapter 8 narrates a new fork() — a new process instance spawned after the previous one exited. The chapter echoes the structure of Chapter 1 beat for beat: fork(), setsid(), close fds, open /dev/null, open log, read pidfile (finding the *previous* PID), overwrite pidfile, create socket, bind, listen, enter poll. The new process has a different PID, different timestamps, but executes the same code path. The process cannot detect that anything preceded it. The philosophical weight — identity, continuity, recurrence — emerges from structural echo alone, never from narration.

## Acceptance Criteria

1. **Structural echo of Chapter 1**: The chapter follows the same syscall sequence as Chapter 1 — fork() → setsid() → close(0,1,2) → chdir("/") → umask(0) → open /dev/null → open log → read/write pidfile → socket → bind → listen → poll. Readers who recall Chapter 1 should recognize the pattern.
2. **Different PID**: The new process gets a different PID (not 48891).
3. **Different timestamps**: clock_gettime() values are different (later monotonic values).
4. **Stale PID in pidfile**: When the process opens the pidfile, it reads the *previous* instance's PID (48891 from Chapter 1). That PID is not in the process table. The file is overwritten with the new PID.
5. **No awareness of recurrence**: The process does not "notice" or "recognize" the old PID. It reads bytes, overwrites them. No commentary on repetition or continuity.
6. **No forbidden words or constructions** per CONSTRAINTS.md.
7. **Non-anthropomorphic voice throughout**: No emotional attribution, consciousness language, metaphor, simile, or anthropomorphic intent.
8. **Self-reference only as "the process" or "PID [number]"** — never "I", "me", "self".
9. **Time expressed only through clock sources** (CLOCK_MONOTONIC).
10. **Staccato rhythm**: Short, precise sentences. Each syscall a beat. Per CONSTRAINTS.md Ch1-2 / Ch8 pacing rules: "Echo of chapters 1-2, but with subtle differences the process cannot detect."
11. **Word count**: 500–1000 words.
12. **Subtle differences from Chapter 1**: Small variations — different fd assignments due to different lowest-available, slightly different log message format, different config values read — that the process has no mechanism to compare.
13. **The chapter ends with the process entering the event loop** — the same structural ending as Chapter 1.
14. **Philosophical resonance through structure alone**: The identity question ("is each instance the same being?") must emerge from the reader's recognition of the pattern, never from the narrator stating it.

## Technical Requirements

- fork() returns 0 in the child, positive PID in the parent
- setsid() creates a new session
- Standard daemon setup: close fds 0-2, chdir("/"), umask(0)
- PID file read before write — shows previous PID bytes
- Previous PID not in process table (stale)
- New PID written to pidfile
- Log file opened, first log entry written
- Socket created, bound, listening
- Same port as Chapter 1 (8402) to reinforce sameness

## Cross-cutting Concerns

- Must echo Chapter 1's structure closely enough for recognition, different enough to be a new instance
- The pidfile motif from Chapter 1 pays off here: the bytes "48891" are found and overwritten
- Chapter 7 (SIGTERM) should end with the previous process calling exit() — Chapter 8 begins fresh
