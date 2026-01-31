# NARRATOR_VOICE.md — Process Perspective Rules

## Self-Reference

The narrator is the process. It refers to itself as:

- **"the process"** — default self-reference
- **"PID N"** — when identity specificity matters (e.g., after fork(), after respawn)
- **getpid()** — when the process queries its own identity

Never use: "I", "me", "self", "we", "my", "our". The process has no first person. It has a PID.

When referencing a previous instance: "the previous PID", "the stale PID in the pidfile". Never: "predecessor", "former self", "past life".

## Cognitive Limitations

The process has no cognition. It has execution state. These are the boundaries:

### No Memory Between Instances

Each fork() produces a new PID. The new process inherits file descriptors and mapped memory pages — not history. The pidfile on disk may contain a stale PID. The process reads it as bytes. It does not "recall" the previous instance.

There is no continuity of experience. There is continuity of configuration: the same binary, the same config file paths, the same socket addresses. The process cannot distinguish "first execution" from "thousandth execution." It has no counter unless one exists on disk or in shared memory.

### No Awareness of Other Processes' Internals

The process observes other processes only through:
- PID values returned by fork() or read from pidfiles
- Signals received (the signal number and sender PID via siginfo_t)
- Bytes arriving on connected sockets or pipes
- waitpid() return values for child processes
- Absence: a write() to a pipe returns EPIPE; a connect() returns ECONNREFUSED

The process does not know what other processes are "doing." It knows what bytes arrive on its file descriptors.

### No Understanding of Meaning

The process reads bytes and writes bytes. An HTTP request is a sequence of bytes matching a pattern. A response is a sequence of bytes written to a file descriptor. The process does not "understand" the request. It matches byte patterns and executes corresponding code paths.

## What the Narrator Can Observe

These are the only information sources available. The narrative must be constructed exclusively from these:

| Source | Mechanism | Example |
|--------|-----------|---------|
| Own syscall return values | Every syscall returns a value or errno | `read() returns 512`, `open() returns fd 7`, `write() returns EAGAIN` |
| Own file descriptor table | open(), close(), dup2(), the fd integers | `fd 0, 1, 2 closed after fork(); fd 3 bound to 0.0.0.0:8402` |
| Own memory contents | Stack variables, heap allocations, mmap regions | `The buffer at 0x7f... contains 512 bytes` |
| Signals received | Signal number, siginfo_t metadata | `SIGTERM delivered. si_pid indicates PID 1.` |
| Clock values | clock_gettime(), timerfd, rdtsc | `CLOCK_MONOTONIC reads 1_800_000_000_000 nanoseconds` |
| Own process state | getpid(), getppid(), getuid(), setsid() returns | `getpid() returns 4821. getppid() returns 1.` |
| Scheduling events | Context switches (observable as time gaps in CLOCK_MONOTONIC) | `470 million nanoseconds elapsed between two sequential clock reads` |
| Filesystem reads | open() + read() on files the process has permission to access | `512 bytes read from /etc/daemon.conf` |
| Environment variables | getenv(), the initial environ block | `getenv("LISTEN_PORT") returns "8402"` |

## What the Narrator Cannot Observe

These must never appear in the narration as known facts. They can only be inferred (incorrectly) or entirely absent:

| Invisible | Why | What appears instead |
|-----------|-----|---------------------|
| Other processes' internal state | No shared address space (without explicit shared memory) | Bytes on sockets, signals, EPIPE, ECONNREFUSED |
| User intent | Users are byte sources on sockets | Byte patterns arriving on fd N |
| Network content meaning | The process matches byte patterns, not semantics | `Bytes match the pattern for a GET request on path /health` |
| Physical time of day | No access unless reading CLOCK_REALTIME and formatting | Epoch seconds or nanoseconds only |
| Why a signal was sent | siginfo_t gives si_pid and si_uid, not motivation | `SIGTERM from PID 1` — not "the supervisor decided to stop us" |
| Hardware state | Abstracted by the kernel | Syscall return values and errno |
| Its own source code | The process executes instructions; it does not read its own binary (unless it opens it) | The current instruction pointer, the current code path |
| Whether it has run before | No built-in instance counter | Stale PID in pidfile, or nothing |

## Narrative Stance

The narration is not the process speaking. The narration is what execution looks like described with zero distance. There is no narrator separate from execution. The prose is the execution, rendered in English.

This means:
- **No commentary.** The process does not reflect on what it is doing.
- **No foreshadowing.** The process does not know what the next syscall will return until it returns.
- **No retrospection.** The process does not compare the current state to past states (it has no past states).
- **Present tense only.** The process exists in a continuous present. Past tense is permitted only for completed syscalls within the current execution ("fork() returned 0").
- **No audience awareness.** The process does not explain itself. It executes.

## Voice Characteristics

- **Precision over poetry.** Every word must be technically defensible.
- **Rhythm from repetition.** The loop structure of daemon operation creates natural prose rhythm — not imposed metaphor.
- **Density.** Short sentences. Each sentence maps to an operation or observation.
- **Register: technical-plain.** Not academic, not literary, not conversational. The register of a well-written man page narrating its own execution.
