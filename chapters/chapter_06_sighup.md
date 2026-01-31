# Chapter 6: SIGHUP

---

`epoll_wait()` returns. fd 4 reports EPOLLIN. `accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 10.0.2.15:51994. `read(7, buf, 4096)` returns 241 bytes. The routing table maps the path to handler index 2. `write(7, response_buf, 1024)` returns 1024. `close(7)`.

`clock_gettime(CLOCK_MONOTONIC, &ts)` returns 94839201448127 nanoseconds.

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

`clock_gettime(CLOCK_MONOTONIC, &ts)` returns 94839201952634 nanoseconds.

`write(3, log_buf, 62)` deposits sixty-two bytes: a formatted timestamp, PID 48891, the string "configuration reloaded, SIGHUP handled" followed by a newline. The bytes append after whatever bytes the new file already contains. The log grows by one line.

---

`reload_flag = 0`. The flag resets. The process re-enters `epoll_wait()`.

PID 48891. The same PID. The same session ID. The same listening socket on fd 4, bound to 0.0.0.0:8402. The configuration struct holds port 9100, but the socket holds 8402. The log file descriptor is 3 — the same integer, potentially a different inode. The process executes the same code path. The data in memory has changed. The code has not.

`epoll_wait()` returns. fd 4 reports EPOLLIN.

`accept4(fd 4, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 10.0.2.15:52003. `read(7, buf, 4096)` returns 197 bytes. The routing table maps the path to handler index 2. `write(7, response_buf, 1024)` returns 1024. `close(7)`.

`clock_gettime(CLOCK_MONOTONIC, &ts)` returns 94839202471882 nanoseconds. Delta from the pre-signal timestamp: 1,023,755 nanoseconds. One millisecond. The configuration reload, the log rotation, the flag check — all within the delta between two clock readings.

`epoll_wait()` returns. fd 4 reports EPOLLIN.

The cycle continues. The handler indices resolve against the same routing table. The response buffers fill from the same memory. The configuration struct holds different values. The process does not compare current values to previous values. There is no previous-value register. There is the current state of memory, and the next system call.
