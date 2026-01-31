# Spec: Chapter 5 — EAGAIN

**Status**: Implemented
**Act**: 2 (The Loop)
**Position**: Final chapter of the main execution cycle
**Word Count Target**: 600–1000 words

## Summary

The process encounters EAGAIN — the errno returned when a non-blocking operation cannot complete immediately. The loop continues but there is nothing to process. `epoll_wait()` returns with no events, or `read()` returns -1 with errno EAGAIN. The chapter captures the process in its idle state: still running, still polling, but with no work arriving. This is the loop without content — pure structure, no payload.

## Technical Foundation

- `epoll_wait()` returning 0 (timeout with no events)
- `read()` / `accept4()` returning -1 with errno set to EAGAIN (or EWOULDBLOCK)
- Non-blocking socket semantics: EAGAIN means "no data available now, try again"
- Process state transitions: TASK_RUNNING → TASK_INTERRUPTIBLE (sleeping in epoll_wait)
- Monotonic clock advancing across empty intervals
- Memory unchanged — no allocations, no frees during idle cycles
- The heap remains static; page tables untouched
- Timer file descriptors or epoll timeouts controlling wake intervals

## Narrative Goals

- Depict the process in its empty loop — executing the same structure with nothing to process
- EAGAIN as the dominant return value: the system call interface saying "nothing here"
- Contrast with the busy accept/read/write/close cycles of Chapter 3
- The process does not experience boredom or waiting — it is simply not scheduled, or it polls and receives EAGAIN
- Time gaps visible only as large deltas between monotonic clock readings
- The prose rhythm should slow down — longer gaps between syscalls, shorter descriptions, more whitespace

## Required Motifs

- **EAGAIN as non-event**: The errno that means "there is nothing" — not an error, not success, just absence
- **The empty loop**: Structure without content; the accept/read/write/close cycle reduced to poll/EAGAIN/poll
- **Clock gaps**: Large monotonic deltas showing time passing with no events — hundreds of milliseconds, then seconds
- **Static memory**: The heap does not grow or shrink; no buffers fill or drain
- **Process state**: TASK_INTERRUPTIBLE — the process yields the CPU, is descheduled, returns only when the kernel wakes it

## Pacing & Rhythm

Per CONSTRAINTS.md: Chapters 3-5 should be "rhythmic, repetitive, meditative."

- The rhythm should decelerate from Chapter 3's busy cycle
- Shorter paragraphs, more section breaks
- Repetition of the same syscall patterns returning nothing
- The prose mirrors the emptiness: sparse, precise, unhurried

## Acceptance Criteria

1. **No forbidden words/constructions** from CONSTRAINTS.md
2. **All process behavior expressed through syscalls/operations only** — no anthropomorphic framing
3. **Technical accuracy**: EAGAIN semantics correct (non-blocking socket, errno, retry semantics)
4. **The process never "waits" in a conscious sense** — it is descheduled or it polls
5. **Monotonic clock** used for all time references, with visible gaps growing larger
6. **At least 3 EAGAIN occurrences** depicted explicitly
7. **Word count**: 600–1000 words
8. **Self-reference**: "the process" only — no "I", "me"
9. **No metaphor or simile**
10. **Contrast with Chapter 3**: the same loop structure, but empty
11. **Process state transitions** mentioned (TASK_INTERRUPTIBLE, scheduling)
12. **Static memory state** noted — no allocations during idle
