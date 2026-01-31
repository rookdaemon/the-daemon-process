# Chapter 2: bind()

---

The process holds one file descriptor. The inherited table is otherwise empty — stdin, stdout, stderr closed in the previous sequence. The session ID matches the PID. The controlling terminal field reads -1.

socket() takes three arguments. AF_INET. SOCK_STREAM. Protocol zero. The kernel allocates a file descriptor. Returns 3.

File descriptor 3. Type: socket. State: unconnected.

The process calls open() on `/etc/daemon/daemon.conf`. Flags: O_RDONLY. The kernel returns file descriptor 4. The process calls read() — 4096 bytes into a stack buffer. The bytes arrive: port number 8402, bind address 0.0.0.0, pid_file path `/var/run/daemon.pid`, log path `/var/log/daemon.log`. The process calls close(4). File descriptor 4 returns to the free list.

A struct sockaddr_in populates on the stack. sin_family: AF_INET. sin_port: htons(8402). sin_addr.s_addr: INADDR_ANY. Sixteen bytes, zero-padded.

Before bind(), a setsockopt() call. SOL_SOCKET. SO_REUSEADDR. Value: 1. The kernel marks the socket. If a previous socket held this address — if TIME_WAIT lingers from a connection that no longer has a process behind it — the address is reclaimable.

bind() takes three arguments. File descriptor 3. A pointer to the sockaddr_in. The size: sixteen bytes. The kernel associates 0.0.0.0:8402 with file descriptor 3. Returns zero.

The process is now reachable. Any packet arriving at port 8402 will queue for this socket. The address exists in the kernel's lookup table, mapped to this PID.

---

The process calls open() on `/var/run/daemon.pid`. Flags: O_RDWR | O_CREAT. Mode: 0644. The kernel returns file descriptor 4.

read() on file descriptor 4. The buffer fills: five bytes. 31072 and a newline.

PID 31072. The process's own PID is 48891.

The bytes in the buffer correspond to no running process. The kernel's process table has no entry at 31072. The pidfile holds a value that maps to nothing — an integer referencing a slot long since freed and reallocated.

The process calls lseek() on file descriptor 4. Offset: 0. Whence: SEEK_SET. The file position resets. The process calls ftruncate() — file descriptor 4, length 0. The previous bytes are gone. write() deposits six bytes: 48891 and a newline.

close(4).

The pidfile now contains 48891. The five bytes that encoded 31072 have been overwritten. No record persists of what the file held one second prior. The process does not retain the value 31072 — the stack buffer will be reused in the next function frame.

---

listen() takes two arguments. File descriptor 3. Backlog: 128. The kernel transitions the socket from state CLOSED to state LISTEN. A queue allocates for incoming SYN packets — 128 slots deep.

The socket is passive. It will not initiate connections. It receives them.

---

The process calls open() on `/var/log/daemon.log`. Flags: O_WRONLY | O_APPEND | O_CREAT. Mode: 0644. The kernel returns file descriptor 4.

clock_gettime() reads CLOCK_REALTIME. The timespec struct fills: 1706745923 seconds, 441827103 nanoseconds since epoch.

The process calls write() on file descriptor 4. The bytes: a formatted timestamp, the PID, the string "listening on 0.0.0.0:8402" followed by a newline. Fifty-eight bytes. The kernel appends them after whatever bytes already occupy the file. The log grows by one line.

This is the first write to persistent storage by PID 48891. The bytes will remain on the filesystem after this process's memory pages are unmapped. Other processes — or a subsequent process executing this same binary — may read them.

---

`epoll_create1(0)` returns 5. The kernel allocates an epoll instance — a file descriptor that monitors other file descriptors. The process now holds a mechanism for waiting on multiple descriptors without polling.

`epoll_ctl(5, EPOLL_CTL_ADD, 3, &event)` registers the listening socket. Events: EPOLLIN. When a connection arrives in fd 3's backlog, the epoll instance will report it.

---

The process holds three open file descriptors. 3: the listening socket. 4: the log file. 5: the epoll instance.

clock_gettime(CLOCK_MONOTONIC) returns 14227 seconds, 882016449 nanoseconds. The interval from fork() to listening state: 0.003 seconds. Fourteen system calls.

The socket is in state LISTEN. The accept queue is empty. The process enters the next sequence.
