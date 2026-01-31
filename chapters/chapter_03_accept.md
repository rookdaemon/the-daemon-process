# Chapter 3: accept()

---

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
