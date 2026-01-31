# Domain Reference: Unix Process Lifecycle

## fork()

- System call that creates a new process by duplicating the calling process
- Returns 0 to the child process, child's PID to the parent
- Child inherits: open file descriptors, signal dispositions, environment, working directory, umask, resource limits
- Child gets: new PID, new memory space (copy-on-write), ppid = parent's PID
- Child does NOT inherit: file locks, pending signals, timers

## Classic Daemon Setup (double-fork pattern)

1. Parent calls fork()
2. Parent exits (child is now orphan, adopted by init/PID 1)
3. Child calls setsid() — becomes session leader, no controlling terminal
4. (Optional) Second fork() — ensures process can never acquire a controlling terminal
5. chdir("/") — avoid holding mount points busy
6. umask(0) — ensure file creation permissions are not restricted
7. Close inherited file descriptors: close(0), close(1), close(2)
8. Open /dev/null on fds 0, 1, 2 (or open log files)
9. Write PID to pidfile (e.g., /var/run/daemon.pid)

## setsid()

- Creates a new session
- Calling process becomes session leader and process group leader
- Process has no controlling terminal after setsid()
- Returns new session ID (same as calling process's PID)

## File Descriptors

- Integer handles to open files, sockets, pipes
- Inherited across fork() — child gets copies
- 0 = stdin, 1 = stdout, 2 = stderr
- Daemons close these to detach from terminal
- New fds assigned as lowest available integer

## PID Files

- Written to /var/run/<name>.pid
- Contains the process's PID as ASCII text
- Used by supervisors and init scripts to track the daemon
- Stale pidfile = previous instance exited without cleanup

## Signals

- SIGTERM: graceful shutdown request
- SIGHUP: traditionally "reload configuration"
- SIGKILL: immediate termination (cannot be caught)
- SIGCHLD: child process state change

## Process States

- R (running/runnable)
- S (interruptible sleep)
- D (uninterruptible sleep)
- Z (zombie — exited but not reaped)
- T (stopped)

## Clock Sources

- CLOCK_MONOTONIC: nanoseconds since arbitrary epoch, never adjusted
- CLOCK_REALTIME: wall clock time (epoch seconds + nanoseconds)
- clock_gettime(clockid, &timespec): retrieves time
- struct timespec { time_t tv_sec; long tv_nsec; }

## Memory

- Virtual address space: text, data, BSS, heap, stack
- brk/sbrk: adjust heap boundary
- mmap: map files or anonymous memory
- Pages: 4096 bytes (typical)
- Copy-on-write after fork: pages shared until written
