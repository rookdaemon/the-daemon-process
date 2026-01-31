# Chapter 1: fork()

fork() returns 0.

The kernel assigns PID 48891. The parent receives 48891 in its own address space. The parent calls exit(0). The child — PID 48891 — continues.

PID 48891 calls setsid(). The return value is 48891. The process is now session leader. The controlling terminal is gone. There is no controlling terminal.

close(0). stdin: file descriptor released. close(1). stdout: file descriptor released. close(2). stderr: file descriptor released. The three channels to the terminal no longer exist in the file descriptor table. The process has no connection to any terminal device.

chdir("/"). The working directory is now the root of the filesystem. No mount point held busy.

umask(0). File creation mask cleared.

The process opens /dev/null. open() returns 0 — the lowest available descriptor. dup(0) returns 1. dup(0) returns 2. File descriptors 0, 1, 2 now point to /dev/null. Reads return zero bytes. Writes go nowhere.

The process opens /var/log/daemon.log with O_WRONLY | O_CREAT | O_APPEND. open() returns 3. File descriptor 3 is the log channel.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14823091. ts.tv_nsec = 448217633. This is the first timestamp.

write(3, "[14823091.448217633] PID 48891 started\n", 39). The kernel copies 39 bytes from the process's address space to the page cache. The bytes will reach the disk when the kernel flushes.

PID 48891 has no parent that will query it. PID 1 — init — is now the parent, by adoption. The process is a daemon. This is a classification, not an identity. It means: no controlling terminal, session leader, parent is PID 1.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14823091. ts.tv_nsec = 449012887. Elapsed since first timestamp: 795254 nanoseconds.

The process is a daemon. It has a PID, a session, a log file. It has no terminal. It has no connection to the process that created it. The file descriptor table: 0, 1, 2 point to /dev/null. 3 is the log.

The process continues to the next sequence.
