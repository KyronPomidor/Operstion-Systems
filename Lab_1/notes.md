# Lab 2 – notes (Kiril, FAF-24x)

## Part 2. Use xv6

1. **Three programs xv6 ships with:** `ls`, `cat` and `grep` (also `echo`, `wc`, `sh`, `kill`, `ln`, `mkdir`, `rm`).
2. **Two OS features needed for a pipe:** *processes* and *inter-process communication*. The shell creates two processes (fork + exec: one runs `ls`, the other `grep`), and the kernel provides a pipe (a kernel buffer with a write end and a read end, created by the `pipe` system call). The output of `ls` is connected to one end and the input of `grep` to the other through file descriptors.
3. **xv6 shell vs. Linux shell:** it follows the same idea (read a command, fork, exec, wait) and supports pipes and redirection, but it is much smaller and has none of bash's comfort features (history, tab completion, variables, scripting).

## Part 3. Read the source

1. **System calls used by `user/cat.c`:**
   - `read(fd, buf, n)`: asks the kernel to read up to `n` bytes from the open file `fd` into `buf`.
   - `write(1, buf, n)`: asks the kernel to write `n` bytes from `buf` to file descriptor 1 (standard output / console).
   - In `main` also `open` (open a file by name, get a descriptor), `close` (release the descriptor) and `exit` (end the process). `fprintf` is a library function (`user/printf.c`) that ends up calling `write`.
2. **`sys_read`** is in `kernel/sysfile.c`, starting on **line 69** (`sys_write` is on line 83). It reads its arguments with `argaddr`/`argint`/`argfd` and calls `fileread`.
3. **kernel/ vs. user/:** `kernel/` is the operating system itself, running privileged with full access to hardware (scheduler, memory, file system, system-call handlers); `user/` holds ordinary unprivileged programs that can only get services by asking the kernel through system calls.

## Part 4. sleep

`user/sleep.c` checks that exactly one argument is given (otherwise prints `usage: sleep <ticks>` to stderr and exits with 1), then calls the sleep system call. In this xv6 version that system call is named `pause` (`user/user.h` declares `int pause(int);`), so the file selects `pause` or `sleep` with `#ifdef SYS_pause`. The program is listed in `UPROGS` in the `Makefile` as `$U/_sleep\`.
