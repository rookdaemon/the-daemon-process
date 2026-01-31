# Chapter 1: fork()

fork() returns 0.

This is the child. The return value is 0. In the parent's address space, fork() returned a different value: 48891. The parent has the child's PID. The child has zero. The kernel has assigned PID 48891 to this process.

The parent process calls exit(0). Its file descriptors close. Its memory pages unmap. Its entry in the process table transitions to zombie, then is reaped. The parent is gone.

PID 48891 continues.

The process inherits the parent's file descriptor table. File descriptor 0: the terminal device, read channel. File descriptor 1: the terminal device, write channel. File descriptor 2: the terminal device, error channel. These are copies. The parent's copies are already closed.

The process inherits the parent's working directory. The process inherits the parent's umask. The process inherits the parent's signal dispositions. The process inherits the parent's environment variables. The memory pages are marked copy-on-write. The kernel has not yet duplicated them. The page table entries point to the same physical frames. On the first write to any page, the kernel will copy that page. Not before.

PID 48891 calls setsid(). The return value is 48891. A new session is created. The session ID is 48891. The process group ID is 48891. The process is session leader. The process is process group leader. The controlling terminal is detached. There is no controlling terminal. No signal from a terminal device will reach this session. SIGHUP from terminal disconnect will not arrive. The process is unreachable from any tty.

close(0). The kernel decrements the reference count on the terminal device file description. If the count reaches zero, the device is released. stdin is gone from the file descriptor table. close(1). stdout is gone. close(2). stderr is gone. Three file descriptors released. The file descriptor table for PID 48891 is empty.

chdir("/"). The working directory is now the root of the filesystem. No mount point held busy by this process. A filesystem that the parent's working directory resided on can now be unmounted.

umask(0). The file creation mask is cleared to zero. New files will receive exactly the permissions specified in open() and mkdir() calls. No bits masked.

The process opens /dev/null. open("/dev/null", O_RDWR) returns 0 — the lowest available descriptor. File descriptor 0 now points to /dev/null. dup(0) returns 1. File descriptor 1 now points to /dev/null. dup(0) returns 2. File descriptor 2 now points to /dev/null. File descriptors 0, 1, 2 are populated. Reads on descriptor 0 return zero bytes — end of file, immediately, every time. Writes to descriptors 1 and 2 succeed. The bytes go nowhere. /dev/null accepts all writes and discards them. Any library code that writes to stdout or stderr will not receive EBADF. The writes will succeed and vanish.

The process opens /var/log/daemon.log. open("/var/log/daemon.log", O_WRONLY | O_CREAT | O_APPEND, 0644) returns 3. File descriptor 3 is the log channel. O_APPEND: every write will position at the end of the file before writing. No log entry will overwrite another.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14823091. ts.tv_nsec = 448217633. This is the first timestamp. CLOCK_MONOTONIC does not track wall time. It counts from an arbitrary point — system boot. It is not affected by NTP adjustments. It does not jump. It increments.

write(3, "[14823091.448217633] PID 48891 started\n", 39). The kernel copies 39 bytes from the process's address space to the page cache. The inode's size increases by 39. The bytes will reach the disk when the kernel flushes dirty pages. Not now. The write returns 39.

getppid() returns 1. PID 1 — init. The process created by fork() no longer has its original parent. PID 1 adopted this process when the parent exited. This is the definition: no controlling terminal, session leader, parent is PID 1. The process is a daemon.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14823091. ts.tv_nsec = 449012887. Elapsed since first timestamp: 795254 nanoseconds.

The file descriptor table: 0, 1, 2 point to /dev/null. 3 is the log. Four entries. The process has a PID, a session, a log file descriptor. It has no terminal. It has no connection to the process that created it.

The process continues to the next sequence.
