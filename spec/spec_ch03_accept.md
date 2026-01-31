# Spec: Chapter 3 — accept()

**Status**: Implemented
**Act**: 2 (The Loop)
**Position**: First chapter of the main execution cycle
**Word Count Target**: 600–1000 words

## Summary

The process enters its main work loop. The listening socket created in Chapter 2 now receives connections via `accept()`. Each connection is a file descriptor — opened, read, written, closed. The rhythm is repetitive, meditative. This is the daemon's purpose: the accept/read/write/close cycle, iterated.

## Technical Foundation

- `epoll_wait()` or `poll()` on the listening socket
- `accept()` returns a new file descriptor for each connection
- `read()` from the client fd, `write()` response back
- `close()` the client fd
- Loop back to `epoll_wait()`
- Connection metadata: source address, port, file descriptor number
- Non-blocking I/O patterns

## Narrative Goals

- Establish the rhythm of the work loop — repetitive, mechanical, meditative
- Each connection is structurally identical but carries different data
- The process does not distinguish "important" from "unimportant" — all bytes are processed the same way
- Time passes only as monotonic clock increments between iterations
- The prose rhythm should mirror the loop: repetitive sentence structures with varying content

## Required Motifs

- **Loop as existence**: The accept/process/close cycle IS the process's entire operational reality
- **Indifference to content**: Bytes flow through; the process routes them without interpretation
- **File descriptor lifecycle**: Each connection is born (accept), used (read/write), and dies (close) — micro-echoes of the process lifecycle
- **Monotonic clock**: Time measured only in nanoseconds between loop iterations

## Pacing & Rhythm

Per CONSTRAINTS.md: Chapters 3-5 should be "rhythmic, repetitive, meditative. The loop structure reflected in prose rhythm."

- Sentences should have a cyclical quality
- Paragraph structure should echo the accept/read/write/close pattern
- Variation comes from the data, not the structure

## Acceptance Criteria

1. **No forbidden words/constructions** from CONSTRAINTS.md
2. **All process behavior expressed through syscalls/operations only** — no anthropomorphic framing
3. **Technical accuracy**: accept(), read(), write(), close(), epoll_wait()/poll() used correctly
4. **The process never "thinks" about connections** — it processes file descriptors
5. **Monotonic clock** used for any time references
6. **Repetitive prose rhythm** reflecting the loop structure
7. **At least 3 complete accept/read/write/close cycles** depicted
8. **Word count**: 600–1000 words
9. **Self-reference**: "the process" or "PID [number]" only — no "I", "me"
10. **File descriptor numbers** mentioned explicitly (e.g., fd 7, fd 8)
11. **Connection source addresses** mentioned (struct sockaddr_in)
12. **No metaphor or simile**
