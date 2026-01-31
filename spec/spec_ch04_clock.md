# Spec: Chapter 4 — clock_gettime()

**Status**: Implemented
**Output**: `chapters/chapter_04_clock.md`

## Summary

Chapter 4 explores the process's relationship with time, expressed entirely through clock_gettime() calls and timer mechanisms. The monotonic clock advances. Intervals accumulate. The process measures durations between events — connections accepted, bytes transferred, timeouts expired — but there is no narrative of time "passing." There is only the delta between two timespec values. This chapter sits at the center of the loop (Act 2) and should feel meditative and repetitive, the rhythm of the event loop reflected in the prose.

## Acceptance Criteria

1. **clock_gettime() is the central mechanism**: Multiple calls to clock_gettime(CLOCK_MONOTONIC, &ts) structure the chapter, each returning increasing tv_sec/tv_nsec values
2. **No forbidden words or constructions** per CONSTRAINTS.md (all grep patterns clean)
3. **Non-anthropomorphic voice throughout**: No emotional attribution, consciousness language, metaphor, simile, or anthropomorphic intent
4. **Self-reference only as "the process" or "PID [number]"** — never "I", "me", "self"
5. **Time expressed only through clock sources**: CLOCK_MONOTONIC, CLOCK_REALTIME, epoch seconds, nanoseconds, timerfd. No human temporal concepts
6. **Meditative, repetitive rhythm**: Per CONSTRAINTS.md Ch3-5 pacing — the loop structure reflected in prose. Repetition as structure, not redundancy
7. **Word count**: 500–1000 words
8. **Timer operations shown**: timerfd_create, timerfd_settime, or poll() timeouts — the process uses timers to schedule periodic work
9. **Monotonic vs realtime distinction**: Show at least one use of both CLOCK_MONOTONIC (for intervals) and CLOCK_REALTIME (for log timestamps or external reporting)
10. **Duration calculation**: The process computes elapsed nanoseconds between events — this is the closest it comes to "experiencing" time
11. **Continuity with chapters 1-3**: The process is the same PID 7291, the same socket on fd 4, the same log on fd 3

## Technical Requirements

- clock_gettime(CLOCK_MONOTONIC, &ts) returns struct timespec {tv_sec, tv_nsec}
- clock_gettime(CLOCK_REALTIME, &ts) returns wall-clock time as epoch seconds
- timerfd_create(CLOCK_MONOTONIC, 0) creates a timer file descriptor
- timerfd_settime() arms the timer with an interval
- poll() or epoll_wait() can use timeouts in milliseconds
- Timer expiry delivers readable events on the timerfd

## Cross-cutting Concerns

- Chapter 8 (Respawn) will show a different monotonic clock base — the new process's CLOCK_MONOTONIC starts from a different kernel-assigned origin, making the values from this chapter unreachable
- The accumulation of nanoseconds here contrasts with the reset in Chapter 8
- Log timestamps (CLOCK_REALTIME) persist on disk and will be readable by the next instance
