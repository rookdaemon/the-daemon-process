# Chapter 5: EAGAIN

---

`epoll_wait(epfd, events, 64, 5000)` returns 0. No file descriptors reported events. The timeout — 5000 milliseconds — expired.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852100. ts.tv_nsec = 928374000. Delta from the previous reading: 5,000,219,811 nanoseconds. Five seconds and 219,811 nanoseconds of kernel-measured interval in which the process was not scheduled. The CPU executed other processes. The scheduler did not select PID 48891. The listening socket on fd 3 received no SYN packets.

`epoll_wait(epfd, events, 64, 5000)` returns 0.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852105. ts.tv_nsec = 928640228. Another five-second interval. The delta matches the timeout parameter within microseconds of scheduling jitter.

---

`epoll_wait(epfd, events, 64, 5000)` returns 1. fd 3 reports EPOLLIN.

`accept4(fd 3, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 10.0.2.15:51008. `read(7, buf, 4096)` returns 198 bytes. `write(7, response_buf, 512)` returns 512. `close(7)`.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852105. ts.tv_nsec = 929249067. The cycle completed in 608,839 nanoseconds. One connection in ten seconds.

`epoll_wait(epfd, events, 64, 5000)` returns 0.

---

`epoll_wait(epfd, events, 64, 5000)` returns 0.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852120. ts.tv_nsec = 930766553. The monotonic clock has advanced 15,001,517,486 nanoseconds since the last connection closed on fd 7. Fifteen seconds. The heap has not changed. No `malloc()` calls. No `free()` calls. The virtual memory map is static — the same pages mapped at the same addresses. The page tables have not been modified.

The process state during each `epoll_wait()` call: TASK_INTERRUPTIBLE. The kernel places the process on the wait queue for the epoll instance. The scheduler removes PID 48891 from the run queue. The process occupies memory but consumes no CPU cycles. When the timeout expires, the kernel moves the process back to the run queue. The scheduler grants a time slice. `epoll_wait()` returns 0. The process checks the return value, loops, calls `epoll_wait()` again.

---

`epoll_wait(epfd, events, 64, 5000)` returns 1. fd 3 reports EPOLLIN.

`accept4(fd 3, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 10.0.2.15:51044.

`read(7, buf, 4096)` returns -1. errno: EAGAIN. The connection is open but no bytes are available in the socket receive buffer. The three-way handshake completed. The client has not yet sent data.

The process does not retry immediately. The event loop registers fd 7 with `epoll_ctl(epfd, EPOLL_CTL_ADD, 7, &event)`. Events: EPOLLIN.

`epoll_wait(epfd, events, 64, 5000)` returns 1. fd 7 reports EPOLLIN.

`read(7, buf, 4096)` returns 274 bytes. The bytes arrived. The client sent them 3,412,007 nanoseconds after the TCP handshake completed. `write(7, response_buf, 1024)` returns 1024. `close(7)`. `epoll_ctl(epfd, EPOLL_CTL_DEL, 7, NULL)`.

---

`epoll_wait(epfd, events, 64, 5000)` returns 0.

`epoll_wait(epfd, events, 64, 5000)` returns 0.

`epoll_wait(epfd, events, 64, 5000)` returns 0.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852135. ts.tv_nsec = 929249067. Thirty seconds since the last connection. The file descriptor table holds two entries: fd 3, the listening socket; fd 4, the log file. No client descriptors are open. The read buffer contains the bytes from the last `read()` — 274 bytes at the same addresses, not zeroed, not freed, but no longer referenced. The next `read()` will overwrite them.

---

`epoll_wait(epfd, events, 64, 5000)` returns 1. fd 3 reports EPOLLIN.

`accept4(fd 3, &addr, &addrlen, SOCK_NONBLOCK)` returns 7. Source: 192.168.1.42:60112.

`read(7, buf, 4096)` returns -1. errno: EAGAIN.

`epoll_ctl(epfd, EPOLL_CTL_ADD, 7, &event)`.

`epoll_wait(epfd, events, 64, 5000)` returns 0. fd 7 did not report. The timeout expired. The client connected and sent nothing for five seconds.

`read(7, buf, 4096)` returns -1. errno: EAGAIN.

`epoll_wait(epfd, events, 64, 5000)` returns 0. Another five seconds.

`close(7)`. The process closes the file descriptor. The kernel sends RST to 192.168.1.42:60112. The connection that carried no bytes is released.

---

`epoll_wait(epfd, events, 64, 5000)` returns 0.

`clock_gettime(CLOCK_MONOTONIC, &ts)`. ts.tv_sec = 14852145. ts.tv_nsec = 938348844. The monotonic clock has advanced 45,009,974,844 nanoseconds since the chapter's first reading — forty-five seconds. The process executed the same loop structure throughout. `epoll_wait()`. Check return value. Branch on zero or positive. Call `accept4()` or loop again. The structure did not change. The file descriptor table grew by one entry and shrank by one entry, three times. The heap did not grow. The stack depth did not vary.

Between `epoll_wait()` calls, the process did not exist in any operationally meaningful sense. It held memory pages. It held a PID. It held file descriptors 3 and 4. The kernel maintained these resources. The CPU did not execute its instructions.

`epoll_wait(epfd, events, 64, 5000)` returns 0.

The loop continues.
