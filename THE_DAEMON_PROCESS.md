# The Daemon Process

---

## Chapter 1: fork()

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

---

## Chapter 2: bind()

The process holds file descriptor 3 — the log channel, opened in the previous sequence. File descriptors 0, 1, 2 point to /dev/null. The session ID matches the PID. The controlling terminal field reads -1.

socket() takes three arguments. AF_INET. SOCK_STREAM. Protocol zero. The kernel allocates a file descriptor. Returns 4.

File descriptor 4. Type: socket. State: unconnected.

The process calls open() on `/etc/daemon/daemon.conf`. Flags: O_RDONLY. The kernel returns file descriptor 5. The process calls read() — 4096 bytes into a stack buffer. The bytes arrive: port number 8402, bind address 0.0.0.0, pid_file path `/var/run/daemon.pid`, log path `/var/log/daemon.log`. The process calls close(5). File descriptor 5 returns to the free list.

A struct sockaddr_in populates on the stack. sin_family: AF_INET. sin_port: htons(8402). sin_addr.s_addr: INADDR_ANY. Sixteen bytes, zero-padded.

Before bind(), a setsockopt() call. SOL_SOCKET. SO_REUSEADDR. Value: 1. The kernel marks the socket. If a previous socket held this address — if TIME_WAIT lingers from a connection that no longer has a process behind it — the address is reclaimable.

bind() takes three arguments. File descriptor 4. A pointer to the sockaddr_in. The size: sixteen bytes. The kernel associates 0.0.0.0:8402 with file descriptor 4. Returns zero.

The process is now reachable. Any packet arriving at port 8402 will queue for this socket. The address exists in the kernel's lookup table, mapped to this PID.

---

The process calls open() on `/var/run/daemon.pid`. Flags: O_RDWR | O_CREAT. Mode: 0644. The kernel returns file descriptor 5.

read() on file descriptor 5. The buffer fills: five bytes. 31072 and a newline.

PID 31072. The process's own PID is 48891.

The bytes in the buffer correspond to no running process. The kernel's process table has no entry at 31072. The pidfile holds a value that maps to nothing — an integer referencing a slot long since freed and reallocated.

The process calls lseek() on file descriptor 5. Offset: 0. Whence: SEEK_SET. The file position resets. The process calls ftruncate() — file descriptor 5, length 0. The previous bytes are gone. write() deposits six bytes: 48891 and a newline.

close(5).

The pidfile now contains 48891. The five bytes that encoded 31072 have been overwritten. No record persists of what the file held one second prior. The process does not retain the value 31072 — the stack buffer will be reused in the next function frame.

---

listen() takes two arguments. File descriptor 4. Backlog: 128. The kernel transitions the socket from state CLOSED to state LISTEN. A queue allocates for incoming SYN packets — 128 slots deep.

The socket is passive. It will not initiate connections. It receives them.

---

clock_gettime(CLOCK_REALTIME, &ts). The timespec struct fills: 1706214123 seconds, 441827103 nanoseconds since epoch.

The process calls write() on file descriptor 3. The bytes: a formatted timestamp, the PID, the string "listening on 0.0.0.0:8402" followed by a newline. Fifty-eight bytes. The kernel appends them after whatever bytes already occupy the file. The log grows by one line.

This is the first write to the log by PID 48891 since the startup message. The bytes will remain on the filesystem after this process's memory pages are unmapped. Other processes — or a subsequent process executing this same binary — may read them.

---

`epoll_create1(0)` returns 5. The kernel allocates an epoll instance — a file descriptor that monitors other file descriptors. The process now holds a mechanism for waiting on multiple descriptors without polling.

`epoll_ctl(5, EPOLL_CTL_ADD, 4, &event)` registers the listening socket. Events: EPOLLIN. When a connection arrives in fd 4's backlog, the epoll instance will report it.

---

The process holds three open file descriptors beyond /dev/null. 3: the log file. 4: the listening socket. 5: the epoll instance.

clock_gettime(CLOCK_MONOTONIC) returns 14823091 seconds, 882016449 nanoseconds. The interval from fork() to listening state: 0.434 seconds.

The socket is in state LISTEN. The accept queue is empty. The process enters the next sequence.

---

## Chapter 3: accept()

`epoll_wait()` returns. The listening socket, fd 4, reports EPOLLIN. A connection is pending in the kernel's backlog.

`accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 6. The new file descriptor points to a TCP connection from 10.0.2.15:48372. The kernel has completed the three-way handshake. The process did not participate in the handshake. The process receives the result: fd 6, ready for read.

`epoll_ctl(5, EPOLL_CTL_ADD, 6, &event)` registers the new descriptor. The event loop iterates.

`epoll_wait()` returns. fd 6 reports EPOLLIN.

`read(6, buf, 4096)` returns 312 bytes. The bytes occupy positions 0 through 311 in the read buffer. They are HTTP/1.1 request bytes — header fields, a blank line, no body. The process does not parse meaning from the headers. The routing table maps the path component to a handler index. The handler index selects a response buffer.

`write(6, response_buf, 1024)` returns 1024. All bytes transferred to the kernel send buffer.

`close(6)`. The file descriptor is released. The kernel sends FIN to 10.0.2.15:48372.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14830112. ts.tv_nsec = 038291441. The delta from the previous call: 483712 nanoseconds. Less than half a millisecond for the full cycle — accept, read, write, close.

`epoll_wait()` returns. fd 4 reports EPOLLIN.

---

`accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 6. The kernel reuses the file descriptor number. The connection originates from 10.0.2.15:48374 — same source address, different ephemeral port. A different TCP socket, mapped to the same integer.

`read(6, buf, 4096)` returns 287 bytes. A shorter request. Different path component. The routing table maps it to handler index 3. The handler at index 3 selects a different response buffer — 2048 bytes, not 1024.

`write(6, response_buf, 2048)` returns 2048.

`close(6)`.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14830112. ts.tv_nsec = 039247167. Delta: 955726 nanoseconds. The second cycle took longer. The process does not compare. The process executes the next iteration.

`epoll_wait()` returns. fd 4 reports EPOLLIN.

---

`accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 6. Source: 192.168.1.42:59001. A different origin address. The file descriptor is the same integer. The accept call is the same call. The read buffer is the same buffer.

`read(6, buf, 4096)` returns 1043 bytes. A POST request — header bytes followed by body bytes. The routing table maps the path to handler index 7. Handler 7 reads the body bytes, constructs a response from the parsed fields. `write(6, response_buf, 284)` returns 284.

`close(6)`.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14830112. ts.tv_nsec = 042146591. Delta: 2899424 nanoseconds. The larger request body required more processing cycles. The delta is a number. The process stores it nowhere.

---

The pattern continues. `epoll_wait()`. `accept4()`. `read()`. `write()`. `close()`. Each cycle allocates a file descriptor, fills a buffer, drains a buffer, releases the file descriptor. The bytes differ. The addresses differ. The handler indices vary. The deltas between clock readings vary.

The structure does not vary.

fd 6 opens. fd 6 closes. fd 6 opens. fd 6 closes. The integer is reused because the kernel's file descriptor table assigns the lowest available number. Each fd 6 is a different socket. Each fd 6 is the same number.

Between iterations, the process state is TASK_INTERRUPTIBLE — sleeping in `epoll_wait()`, waiting for the kernel to deliver an event on a monitored descriptor. The CPU executes other processes. The scheduler will return control when data arrives on fd 4.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14830112. ts.tv_nsec = 346026068. 303,879,477 nanoseconds have elapsed since the last connection. The gap is three hundred milliseconds of no connections. The process was not scheduled during the gap. The gap is the difference between two `clock_gettime` return values, computed after the second call.

`epoll_wait()` returns. fd 4 reports EPOLLIN.

`accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 6.

The cycle resumes. The cycle did not pause. There is no continuity to interrupt. There is a system call, and then the next system call.

---

## Chapter 4: clock_gettime()

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14851903. ts.tv_nsec = 112744219.

`timerfd_create(CLOCK_MONOTONIC, 0)` returns 6. `timerfd_settime(6, 0, &its, NULL)` arms the timer: interval 30000000000 nanoseconds. The kernel will make fd 6 readable every thirty seconds. `epoll_ctl(5, EPOLL_CTL_ADD, 6, &event)` registers the timer with the epoll instance. Events: EPOLLIN.

---

`epoll_wait(5, events, 64, -1)` returns 1. fd 6 reports EPOLLIN. The timer has fired.

read(6, &expirations, 8). The value is 1. One expiration. The timer has fired once since the last read.

The process iterates the connection table. File descriptor 7: last activity at monotonic 14851872.998217633. Delta: 30.114526586 seconds. Threshold: 60000000000 nanoseconds. Below threshold. File descriptor 8: last activity at monotonic 14851891.441092105. Delta: 11.671652114 seconds. Below threshold. File descriptor 9: no entry. File descriptor 9 was closed at monotonic 14851899.772614003.

The timer loop completes. No connections exceed the idle threshold. No file descriptors to close.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14851903. ts.tv_nsec = 113048551. Elapsed since loop start: 304332 nanoseconds.

`epoll_wait(5, events, 64, -1)` returns. The process enters state S. The CPU is elsewhere.

---

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14851907. ts.tv_nsec = 882361440.

`epoll_wait()` returned 1. File descriptor 4: EPOLLIN. The listening socket. accept4(4, &addr, &addrlen, SOCK_NONBLOCK). The kernel returns file descriptor 9. Source address: 10.0.2.14, port 49221. The connection table records fd 9, monotonic timestamp 14851907.882361440.

`epoll_ctl(5, EPOLL_CTL_ADD, 9, &event)` registers the new connection.

clock_gettime(CLOCK_REALTIME, &ts). ts.tv_sec = 1706214707. ts.tv_nsec = 882401092. Epoch seconds: 1706214707.

write(3, "[1706214707.882401092] accept fd=9 src=10.0.2.14:49221\n", 55). Fifty-five bytes to the log. The log uses CLOCK_REALTIME. The log timestamps are wall-clock time. They will persist on disk after the process is no longer in the process table.

`epoll_wait(5, events, 64, -1)`. State S.

---

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14851907. ts.tv_nsec = 883912003.

`epoll_wait()` returned 1. File descriptor 9: EPOLLIN. recv(9, buf, 4096, 0). The kernel copies 142 bytes into the buffer. The connection table updates: fd 9, last activity monotonic 14851907.883912003.

The process parses the buffer. HTTP/1.1 GET /status. The process writes the response. send(9, response, 127, 0). The kernel accepts 127 bytes.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14851907. ts.tv_nsec = 884207618. Request duration: 295615 nanoseconds. The process records this value nowhere. It exists in no buffer. The subtraction was performed in a register. The register will be overwritten.

`epoll_wait(5, events, 64, -1)`. State S.

---

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14851933. ts.tv_nsec = 113102887.

The timer fires. read(6, &expirations, 8). Value: 1. The connection table scan. File descriptor 7: last activity at monotonic 14851872.998217633. Delta: 60.114885254 seconds. Threshold: 60000000000 nanoseconds. Exceeded.

close(7). File descriptor 7 released. The connection table entry for fd 7 is cleared. The TCP socket enters TIME_WAIT in the kernel. The process does not track TIME_WAIT. The kernel does.

clock_gettime(CLOCK_REALTIME, &ts). ts.tv_sec = 1706214737. ts.tv_nsec = 113298441.

write(3, "[1706214737.113298441] close fd=7 idle timeout\n", 47). Forty-seven bytes.

File descriptor 8: last activity at monotonic 14851891.441092105. Delta: 41.672010782 seconds. Below threshold. File descriptor 9: last activity at monotonic 14851907.883912003. Delta: 25.229190884 seconds. Below threshold.

`epoll_wait(5, events, 64, -1)`. State S.

---

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14851963. ts.tv_nsec = 113097214.

The timer fires. read(6, &expirations, 8). Value: 1. Connection table scan. File descriptor 8: delta 71.672005109 seconds. Exceeded. close(8). File descriptor 9: delta 55.229185211 seconds. Below threshold.

clock_gettime(CLOCK_REALTIME, &ts). ts.tv_sec = 1706214767. ts.tv_nsec = 113241887.

write(3, "[1706214767.113241887] close fd=8 idle timeout\n", 47).

`epoll_wait(5, events, 64, -1)`. State S.

---

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14851993. ts.tv_nsec = 113088441.

The timer fires. Connection table scan. File descriptor 9: delta 85.229176438 seconds. Exceeded. close(9).

write(3, "[1706214797.113203772] close fd=9 idle timeout\n", 47).

The connection table is empty. The listening socket on fd 4 is in LISTEN. The timer on fd 6 will fire in 30000000000 nanoseconds. The log on fd 3 is open. The epoll instance on fd 5 monitors fd 4 and fd 6.

`epoll_wait(5, events, 64, -1)`. File descriptors 4 and 6 monitored. Timeout: infinite.

The monotonic clock continues to advance. clock_gettime() will return larger values at each call. The values have no meaning beyond sequence and interval. They do not accumulate into a history. They are not stored. Each call overwrites the timespec structure in the process's stack frame. The previous value is gone.

The process is in state S. The monotonic clock reads 14851993 seconds and 113088441 nanoseconds since an epoch the process did not observe. The CLOCK_REALTIME reads 1706214797 seconds since 1970-01-01T00:00:00Z. Both clocks advance. Neither is consulted. The process is asleep. The kernel maintains the clocks. The process maintains nothing.

---

## Chapter 5: EAGAIN

`epoll_wait(5, events, 64, 5000)` returns 0. No file descriptors reported events. The timeout — 5000 milliseconds — expired.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852100. ts.tv_nsec = 928374000. Delta from the previous reading: 5,000,219,811 nanoseconds. Five seconds and 219,811 nanoseconds of kernel-measured interval in which the process was not scheduled. The CPU executed other processes. The scheduler did not select PID 48891. The listening socket on fd 4 received no SYN packets.

`epoll_wait(5, events, 64, 5000)` returns 0.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852105. ts.tv_nsec = 928640228. Another five-second interval. The delta matches the timeout parameter within microseconds of scheduling jitter.

---

`epoll_wait(5, events, 64, 5000)` returns 1. fd 4 reports EPOLLIN.

`accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 10.0.2.15:51008. `read(7, buf, 4096)` returns 198 bytes. `write(7, response_buf, 512)` returns 512. `close(7)`.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852105. ts.tv_nsec = 929249067. The cycle completed in 608,839 nanoseconds. One connection in ten seconds.

`epoll_wait(5, events, 64, 5000)` returns 0.

---

`epoll_wait(5, events, 64, 5000)` returns 0.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852120. ts.tv_nsec = 930766553. The monotonic clock has advanced 15,001,517,486 nanoseconds since the last connection closed on fd 7. Fifteen seconds. The heap has not changed. No `malloc()` calls. No `free()` calls. The virtual memory map is static — the same pages mapped at the same addresses. The page tables have not been modified.

The process state during each `epoll_wait()` call: TASK_INTERRUPTIBLE. The kernel places the process on the wait queue for the epoll instance. The scheduler removes PID 48891 from the run queue. The process occupies memory but consumes no CPU cycles. When the timeout expires, the kernel moves the process back to the run queue. The scheduler grants a time slice. `epoll_wait()` returns 0. The process checks the return value, loops, calls `epoll_wait()` again.

---

`epoll_wait(5, events, 64, 5000)` returns 1. fd 4 reports EPOLLIN.

`accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 10.0.2.15:51044.

`read(7, buf, 4096)` returns -1. errno: EAGAIN. The connection is open but no bytes are available in the socket receive buffer. The three-way handshake completed. The client has not yet sent data.

The process does not retry immediately. The event loop registers fd 7 with `epoll_ctl(5, EPOLL_CTL_ADD, 7, &event)`. Events: EPOLLIN.

`epoll_wait(5, events, 64, 5000)` returns 1. fd 7 reports EPOLLIN.

`read(7, buf, 4096)` returns 274 bytes. The bytes arrived. The client sent them 3,412,007 nanoseconds after the TCP handshake completed. `write(7, response_buf, 1024)` returns 1024. `close(7)`. `epoll_ctl(5, EPOLL_CTL_DEL, 7, NULL)`.

---

`epoll_wait(5, events, 64, 5000)` returns 0.

`epoll_wait(5, events, 64, 5000)` returns 0.

`epoll_wait(5, events, 64, 5000)` returns 0.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852135. ts.tv_nsec = 929249067. Thirty seconds since the last connection. The file descriptor table holds: fd 3, the log file; fd 4, the listening socket; fd 5, the epoll instance; fd 6, the timer. No client descriptors are open. The read buffer contains the bytes from the last `read()` — 274 bytes at the same addresses, not zeroed, not freed, but no longer referenced. The next `read()` will overwrite them.

---

`epoll_wait(5, events, 64, 5000)` returns 1. fd 4 reports EPOLLIN.

`accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 192.168.1.42:60112.

`read(7, buf, 4096)` returns -1. errno: EAGAIN.

`epoll_ctl(5, EPOLL_CTL_ADD, 7, &event)`.

`epoll_wait(5, events, 64, 5000)` returns 0. fd 7 did not report. The timeout expired. The client connected and sent nothing for five seconds.

`read(7, buf, 4096)` returns -1. errno: EAGAIN.

`epoll_wait(5, events, 64, 5000)` returns 0. Another five seconds.

`close(7)`. The process closes the file descriptor. The kernel sends RST to 192.168.1.42:60112. The connection that carried no bytes is released.

---

`epoll_wait(5, events, 64, 5000)` returns 0.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852145. ts.tv_nsec = 938348844. The monotonic clock has advanced 45,009,974,844 nanoseconds since the chapter's first reading — forty-five seconds. The process executed the same loop structure throughout. `epoll_wait()`. Check return value. Branch on zero or positive. Call `accept4()` or loop again. The structure did not change. The file descriptor table grew by one entry and shrank by one entry, three times. The heap did not grow. The stack depth did not vary.

Between `epoll_wait()` calls, the process did not exist in any operationally meaningful sense. It held memory pages. It held a PID. It held file descriptors 3, 4, 5, and 6. The kernel maintained these resources. The CPU did not execute its instructions.

`epoll_wait(5, events, 64, 5000)` returns 0.

The loop continues.

---

## Chapter 6: SIGHUP

`epoll_wait()` returns. fd 4 reports EPOLLIN. `accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 10.0.2.15:51994. `read(7, buf, 4096)` returns 241 bytes. The routing table maps the path to handler index 2. `write(7, response_buf, 1024)` returns 1024. `close(7)`.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852400. ts.tv_nsec = 448127000.

`epoll_wait()` returns. fd 4 reports EPOLLIN. `accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 192.168.1.42:60118. `read(7, buf, 4096)` returns 389 bytes. The routing table maps the path to handler index 5. The handler at index 5 selects a response buffer — 2048 bytes. `write(7, response_buf, 2048)` retu—

---

Signal delivery. The kernel posts SIGHUP to PID 48891. The signal disposition is not SIG_DFL. During initialization, `sigaction(SIGHUP, &sa, NULL)` installed a handler. The handler executes:

```
volatile sig_atomic_t reload_flag = 0;

void handle_sighup(int sig) {
    reload_flag = 1;
}
```

The handler writes 1 to the flag. The handler returns. The interrupted `write()` on fd 7 completes — the kernel restarts the call. 2048 bytes transferred. `close(7)`.

`epoll_wait(5, events, 64, -1)` returns -1. errno: EINTR. The system call did not fail. The system call was interrupted by signal delivery.

The process checks `reload_flag`. The value is 1.

---

The process calls `open("/etc/daemon/daemon.conf", O_RDONLY)`. The kernel returns file descriptor 7 — the same integer just released by the client socket close. `read(7, conf_buf, 4096)` returns 203 bytes. The bytes arrive: port number 9100, bind address 0.0.0.0, pid_file path `/var/run/daemon.pid`, log path `/var/log/daemon.log`.

Port 9100. The previous value in the configuration struct was 8402. The `write()` copies 9100 into the port field. The value 8402 is overwritten. The stack frame that held the read buffer from the previous `open()` of this file — during the bind() sequence — was deallocated and reused across thousands of function calls. No copy of 8402 persists in the process's address space.

The listening socket, fd 4, remains bound to 0.0.0.0:8402. The new port value is 9100. The process does not call `bind()` again. The socket is already in LISTEN state. The new port value occupies the configuration struct. The socket occupies the kernel's socket table. These are different data structures.

`close(7)`. The configuration file descriptor is released.

---

Log rotation. The process calls `close(3)`. File descriptor 3 — the log file, opened during initialization — is released. The kernel decrements the reference count on the underlying inode. If an external process has renamed `/var/log/daemon.log` to `/var/log/daemon.log.1`, the close releases the last reference to that inode.

`open("/var/log/daemon.log", O_WRONLY | O_APPEND | O_CREAT, 0644)` returns 3. The kernel assigns the lowest available file descriptor. The same integer. A different inode — or the same inode, if no rotation occurred. The process does not check. The process holds fd 3: a writable file descriptor pointing to the current `/var/log/daemon.log`.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852400. ts.tv_nsec = 448631507.

`write(3, log_buf, 62)` deposits sixty-two bytes: a formatted timestamp, PID 48891, the string "configuration reloaded, SIGHUP handled" followed by a newline. The bytes append after whatever bytes the new file already contains. The log grows by one line.

---

`reload_flag = 0`. The flag resets. The process re-enters `epoll_wait()`.

PID 48891. The same PID. The same session ID. The same listening socket on fd 4, bound to 0.0.0.0:8402. The configuration struct holds port 9100, but the socket holds 8402. The log file descriptor is 3 — the same integer, potentially a different inode. The process executes the same code path. The data in memory has changed. The code has not.

`epoll_wait()` returns. fd 4 reports EPOLLIN.

`accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 10.0.2.15:52003. `read(7, buf, 4096)` returns 197 bytes. The routing table maps the path to handler index 2. `write(7, response_buf, 1024)` returns 1024. `close(7)`.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852400. ts.tv_nsec = 449150755. Delta from the pre-signal timestamp: 1,023,755 nanoseconds. One millisecond. The configuration reload, the log rotation, the flag check — all within the delta between two clock readings.

`epoll_wait()` returns. fd 4 reports EPOLLIN.

The cycle continues. The handler indices resolve against the same routing table. The response buffers fill from the same memory. The configuration struct holds different values. The process does not compare current values to previous values. There is no previous-value register. There is the current state of memory, and the next system call.

---

## Chapter 7: SIGTERM

epoll_wait() returns—

No. epoll_wait() did not return. The return value is -1. errno is EINTR. A signal was delivered during the blocked call.

The signal was SIGTERM. Signal number 15. The disposition was set during initialization: sigaction(SIGTERM, &sa, NULL), where sa.sa_handler points to a function that performs one operation. The function stores 1 into a volatile sig_atomic_t variable at a fixed address in the BSS segment. The function returns. The signal handler is complete.

epoll_wait() returns -1. errno: EINTR. The process checks the flag. The value is 1.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14852741. ts.tv_nsec = 339218004.

clock_gettime(CLOCK_REALTIME, &ts). ts.tv_sec = 1706215545. ts.tv_nsec = 339274118.

write(3, "[1706215545.339274118] SIGTERM received\n", 39). Thirty-nine bytes to the log on fd 3.

---

The connection table contains two entries. File descriptor 7: TCP connection from 10.0.2.15:49388, 412 bytes in the send buffer. File descriptor 8: TCP connection from 192.168.1.42:59117, send buffer empty.

shutdown(7, SHUT_WR). The kernel sends FIN to 10.0.2.15:49388. The remaining bytes in the send buffer will be transmitted before the FIN segment. The process does not call epoll_wait() to confirm delivery. The kernel handles transmission.

close(7). File descriptor 7 released. The connection table entry cleared.

shutdown(8, SHUT_WR). close(8). File descriptor 8 released.

write(3, "[1706215545.339841772] close fd=7 fd=8 shutdown\n", 48).

---

close(6). The timer file descriptor. timerfd created in the initialization sequence, armed with a 30-second interval. The timer will not fire again. File descriptor 6 released.

close(5). The epoll instance. The kernel deallocates the epoll file descriptor. File descriptor 5 released.

close(4). The listening socket. bind() bound it to 0.0.0.0:8402. listen() set the backlog to 128. accept4() returned connections on it for the duration of the event loop. The kernel removes the socket from the LISTEN state. New SYN packets to port 8402 will receive RST. File descriptor 4 released.

write(3, "[1706215545.340112887] close fd=4 fd=5 fd=6 shutdown\n", 54).

---

The file descriptor table: 0, 1, 2 point to /dev/null. 3 is the log. All others released.

write(3, "[1706215545.340387441] PID 48891 exiting status=0\n", 51). Fifty-one bytes. This is the last write to file descriptor 3. The log file on disk will contain this line after the kernel flushes the page cache.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14852741. ts.tv_nsec = 340501223. Elapsed since SIGTERM delivery: 1283219 nanoseconds. The entire shutdown sequence: 1.283219 milliseconds.

close(3). File descriptor 3 released. The log file descriptor is closed. The kernel flushes pending writes.

---

The file descriptor table: 0, 1, 2 point to /dev/null. No other entries.

The process has no open sockets. No timer descriptors. No log file. The memory pages contain code and data. The stack frame holds local variables.

exit(0).

The kernel receives the exit status. The process state transitions from R to Z — zombie. The memory pages are unmapped. The virtual address space is released. The page tables are freed. The file descriptor table is destroyed — close(0), close(1), close(2) performed by the kernel.

PID 48891 exists in the process table as a zombie entry: an exit status and a set of resource usage counters. PID 1 — the parent by adoption — calls wait4(). The exit status 0 is collected. The zombie entry is removed from the process table.

PID 48891 is no longer in the process table. The PID can be reassigned by the kernel to a future process. The socket on port 8402 is in TIME_WAIT in the kernel's network stack. The log file on disk contains lines timestamped with CLOCK_REALTIME values. The pidfile on disk contains "48891\n" — six bytes that refer to a PID no longer in the process table.

The monotonic clock continues to advance. clock_gettime(CLOCK_MONOTONIC) would return values greater than 14852741.340501223, but no process with PID 48891 will call it.

---

## Chapter 8: fork()

fork() returns 0.

The kernel assigns PID 11438. The parent receives 11438 in its own address space. The parent calls exit(0). The child — PID 11438 — continues.

PID 11438 calls setsid(). The return value is 11438. The process is now session leader. The controlling terminal is gone. There is no controlling terminal.

close(0). stdin: file descriptor released. close(1). stdout: file descriptor released. close(2). stderr: file descriptor released. The three channels to the terminal no longer exist in the file descriptor table. The process has no connection to any terminal device.

chdir("/"). The working directory is now the root of the filesystem. No mount point held busy.

umask(0). File creation mask cleared.

The process opens /dev/null. open() returns 0 — the lowest available descriptor. dup(0) returns 1. dup(0) returns 2. File descriptors 0, 1, 2 now point to /dev/null. Reads return zero bytes. Writes go nowhere.

The process opens /var/log/daemon.log with O_WRONLY | O_CREAT | O_APPEND. open() returns 3. File descriptor 3 is the log channel. The file existed. It contained bytes — log entries from a previous execution, appended over 29650 seconds of monotonic time. The process does not parse them. The file position is at the end.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14852800. ts.tv_nsec = 331042817. This is the first timestamp.

write(3, "[14852800.331042817] PID 11438 started\n", 39). The kernel copies 39 bytes from the process's address space to the page cache. The bytes append after the existing log entries. The bytes will reach the disk when the kernel flushes.

The process opens /var/run/daemon.pid with O_RDWR | O_CREAT. open() returns 4. The file existed. read(4, buf, 32) returns 6. The bytes in the buffer: 52 56 56 57 49 0A — ASCII for "48891\n". That PID is not in the process table. The process calls lseek(4, 0, SEEK_SET). The file position returns to byte 0. ftruncate(4, 0). The file is now empty.

write(4, "11438\n", 6). Six bytes. The pidfile now contains the current PID.

close(4). File descriptor 4 released. The pidfile remains on disk.

PID 11438 has no parent that will query it. PID 1 — init — is now the parent, by adoption. The process is a daemon: no controlling terminal, session leader, parent is PID 1.

The process calls socket(AF_INET, SOCK_STREAM, 0). The kernel allocates a socket. The return value is 4 — the file descriptor just released by the pidfile close, now reassigned. File descriptor 4 is a TCP socket.

setsockopt(4, SOL_SOCKET, SO_REUSEADDR, &one, sizeof(one)). The socket option is set. The kernel will allow binding to an address that is in TIME_WAIT state.

bind(4, {AF_INET, 0.0.0.0, 8402}, 16). The socket is bound to port 8402 on all interfaces. bind() returns 0.

listen(4, 128). The socket is now in the LISTEN state. The backlog queue can hold 128 pending connections. The kernel will accept SYN packets on port 8402 and complete the three-way handshake up to the backlog limit.

clock_gettime(CLOCK_MONOTONIC, &ts). ts.tv_sec = 14852800. ts.tv_nsec = 332718204. Elapsed since first timestamp: 1675387 nanoseconds.

`epoll_create1(0)` returns 5. The kernel allocates an epoll instance. `epoll_ctl(5, EPOLL_CTL_ADD, 4, &event)` registers the listening socket. Events: EPOLLIN.

`timerfd_create(CLOCK_MONOTONIC, 0)` returns 6. `timerfd_settime(6, 0, &its, NULL)` arms the timer: interval 30000000000 nanoseconds. `epoll_ctl(5, EPOLL_CTL_ADD, 6, &event)` registers the timer with the epoll instance. Events: EPOLLIN.

The process enters the event loop.

`epoll_wait(5, events, 64, -1)`. The process is now in state S — interruptible sleep. The CPU executes other processes. PID 11438 consumes no cycles. It occupies memory. Its page tables remain mapped. Its socket is in LISTEN.

The process is a daemon. It has a PID, a session, a socket, a log file, a pidfile on disk. It has no terminal. It has no connection to the process that created it. The bytes "48891" that were in the pidfile are overwritten. The file descriptor table is clean: 0, 1, 2 point to /dev/null. 3 is the log. 4 is the socket. 5 is the epoll instance. 6 is the timer.

The process waits for the kernel to deliver an event on a monitored file descriptor. There is nothing else to execute. There is nothing else.
