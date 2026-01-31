# CONSTRAINTS.md — Hard Rules for "The Daemon Process"

## Absolute Prohibitions

### Forbidden Constructions (grep-able)

**Emotional attribution — NEVER use:**
- "think", "thought", "thinking"
- "feel", "felt", "feeling"
- "wonder", "wondered"
- "hope", "hoped"
- "want", "wanted", "desire"
- "fear", "afraid", "worry"
- "miss", "missed", "long for"
- "love", "hate", "enjoy"
- "happy", "sad", "lonely", "angry"
- "content", "satisfied", "frustrated"

**Consciousness language — NEVER use:**
- "realize", "understand", "comprehend"
- "know", "knew", "knowledge"
- "remember", "recall", "forget", "memory" (as cognitive concept — "memory" as RAM is fine)
- "decide", "chose", "choice"
- "recognize", "notice", "aware"
- "believe", "doubt", "suspect"
- "imagine", "dream", "wish"
- "experience" (as a verb of consciousness)

**Anthropomorphic intent — NEVER use:**
- "tries to", "attempts to", "wants to", "needs to"
- "refuses to", "insists on"
- "learns", "adapts" (in consciousness sense — adaptive algorithms are fine if technically described)
- "struggles with", "succeeds at"
- "is born", "dies" (use fork/exit — the technical terms ARE the events)

**Metaphor and simile — NEVER use:**
- "as if", "like a", "reminds of"
- "seems to", "appears to", "as though"
- Any comparison to living things, weather, or human activities

**Human temporal concepts — NEVER use:**
- "yesterday", "tomorrow", "soon", "eventually", "later"
- "morning", "evening", "night" (use clock values)
- "young", "old", "age", "aging"
- "before" / "after" in narrative-memory sense (before/after in sequence-of-operations sense is fine)

## Required Approaches

### Process "experience" must be expressed through:
- System calls: fork(), setsid(), open(), read(), write(), close(), exit()
- Signal delivery: SIGTERM, SIGHUP, SIGCHLD, SIGKILL
- Memory operations: mmap, brk, allocation, deallocation, page faults
- File descriptor operations: open, read, write, poll, close
- Clock sources: CLOCK_MONOTONIC, CLOCK_REALTIME, clock_gettime()
- Network operations: bind, listen, accept, recv, send
- Process state: running, sleeping, zombie, stopped
- Scheduling: context switches, time slices, CPU affinity

### Time must be expressed through:
- Monotonic nanoseconds (clock_gettime(CLOCK_MONOTONIC))
- Timer file descriptors (timerfd_create)
- Epoch seconds (time())
- CPU cycles (rdtsc)
- NOT human time concepts

### "Death" is:
- exit(0) or exit(1)
- SIGKILL delivery
- Process state transition to zombie, then reaping by parent
- File descriptors closing
- Memory pages unmapped

### "Birth" is:
- fork() returning 0 in child
- New PID assignment
- setsid() creating new session
- File descriptor inheritance (then closing inherited ones)

### "Memory" is:
- Virtual memory space (mmap, brk)
- Stack frames
- Heap allocations
- Shared memory segments
- NOT recollection or nostalgia

## Terminology Standards

### The Process
- Self-reference: "the process" or "PID [number]"
- NEVER: "I", "me", "self" (unless referring to getpid())
- Previous instance: "the previous PID" or "the stale PID in the pidfile"
- NOT: "predecessor", "ancestor" (too anthropomorphic), "former self"

### System/Environment
- Operating system: "the kernel"
- Hardware: "the CPU", "the NIC", "the disk"
- Other processes: "PID [number]" or "the parent process", "the supervisor"
- NOT: "the world", "the universe", "the outside"

### Events
- Startup: "fork()", "setsid()", "the process begins execution"
- Shutdown: "exit()", "SIGTERM delivery", "file descriptors close"
- NOT: "birth", "death", "life", "end" (too loaded — use technical terms)

## Rhythm and Pacing by Chapter

- **Chapters 1-2 (Fork)**: Short, precise sentences. Each syscall a beat. Staccato.
- **Chapters 3-5 (Loop)**: Rhythmic, repetitive, meditative. The loop structure reflected in prose rhythm.
- **Chapters 6-7 (Signals)**: Interruption. Mid-sentence breaks. The prose itself gets SIGHUPed.
- **Chapter 8 (Respawn)**: Echo of chapters 1-2, but with subtle differences the process cannot detect.
