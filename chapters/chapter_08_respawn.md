# Chapter 8: fork()

fork() returns 0.

The kernel assigns PID 11438. The parent receives 11438 in its own address space. The parent calls exit(0). The child — PID 11438 — continues.

PID 11438 calls setsid(). The return value is 11438. The process is now session leader. The controlling terminal is gone. There is no controlling terminal.

close(0). stdin: file descriptor released. close(1). stdout: file descriptor released. close(2). stderr: file descriptor released. The three channels to the terminal no longer exist in the file descriptor table. The process has no connection to any terminal device.

chdir("/"). The working directory is now the root of the filesystem. No mount point held busy.

umask(0). File creation mask cleared.

The process opens /dev/null. open() returns 0 — the lowest available descriptor. dup(0) returns 1. dup(0) returns 2. File descriptors 0, 1, 2 now point to /dev/null. Reads return zero bytes. Writes go nowhere.

The process opens /var/log/daemon.log with O_WRONLY | O_CREAT | O_APPEND. open() returns 3. File descriptor 3 is the log channel. The file existed. It contained bytes — log entries from a previous execution, appended over 94877 seconds of monotonic time. The process does not parse them. The file position is at the end.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 94891812. ts.tv_nsec = 331042817. This is the first timestamp.

write(3, "[94891812.331042817] PID 11438 started\n", 39). The kernel copies 39 bytes from the process's address space to the page cache. The bytes append after the existing log entries. The bytes will reach the disk when the kernel flushes.

The process opens /var/run/daemon.pid with O_RDWR | O_CREAT. open() returns 4. The file existed. read(4, buf, 32) returns 5. The bytes in the buffer: 55 50 57 49 0A — ASCII for "7291\n". That PID is not in the process table. The process calls lseek(4, 0, SEEK_SET). The file position returns to byte 0. ftruncate(4, 0). The file is now empty.

write(4, "11438\n", 6). Six bytes. The pidfile now contains the current PID.

close(4). File descriptor 4 released. The pidfile remains on disk.

PID 11438 has no parent that will query it. PID 1 — init — is now the parent, by adoption. The process is a daemon. This is a classification, not an identity. It means: no controlling terminal, session leader, parent is PID 1.

The process calls socket(AF_INET, SOCK_STREAM, 0). The kernel allocates a socket. The return value is 4 — the file descriptor just released by the pidfile close, now reassigned. File descriptor 4 is a TCP socket.

setsockopt(4, SOL_SOCKET, SO_REUSEADDR, &one, sizeof(one)). The socket option is set. The kernel will allow binding to an address that is in TIME_WAIT state.

bind(4, {AF_INET, 0.0.0.0, 8080}, 16). The socket is bound to port 8080 on all interfaces. bind() returns 0.

listen(4, 128). The socket is now in the LISTEN state. The backlog queue can hold 128 pending connections. The kernel will accept SYN packets on port 8080 and complete the three-way handshake up to the backlog limit.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 94891812. ts.tv_nsec = 332718204. Elapsed since first timestamp: 1675387 nanoseconds.

The process enters the event loop.

poll(&fds, 1, -1). File descriptor 4. Events: POLLIN. Timeout: infinite. The process is now in state S — interruptible sleep. The CPU executes other processes. PID 11438 consumes no cycles. It occupies memory. Its page tables remain mapped. Its socket is in LISTEN.

The process is a daemon. It has a PID, a session, a socket, a log file, a pidfile on disk. It has no terminal. It has no connection to the process that created it. The bytes "7291" that were in the pidfile are overwritten. The file descriptor table is clean: 0, 1, 2 point to /dev/null. 3 is the log. 4 is the socket.

The process waits for the kernel to deliver an event on file descriptor 4. There is nothing else to execute. There is nothing else.
