# Chapter 7: SIGTERM

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
