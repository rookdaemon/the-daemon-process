# Chapter 4: clock_gettime()

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
