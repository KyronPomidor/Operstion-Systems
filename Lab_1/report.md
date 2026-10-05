# Lab 1 - The OS as a Resource Manager (Linux)

**Student:** Kiril Boboc (FAF-242) 

## Setup
`whoami` → kiril; `uname -a` → Linux macLab 7.0.0-34-generic ... aarch64; `uptime` → up a few minutes, load average 0.15 0.10 0.04 (idle).

## Part 1. Files and directories
Commands: `pwd`, `ls -la /`, `ls -la ~`, `mkdir`, `echo > note.txt`, `cp`, `mv`, `rm`, `ls -l`, `chmod 600/644`.
Interesting output:
```
-rw-rw-r-- 1 kiril kiril 24 Oct  3 19:40 note.txt
-rw------- 1 kiril kiril 24 Oct  3 19:40 note.txt   (after chmod 600)
```
(Note: `mkdir demo` said "File exists", so the files were created in ~/os-lab1.)
1. Owner **kiril**, group **kiril** (the OS gives a new file the owner and primary group of the creating user).
2. `-rw-------`: `-` = regular file; `rw-` owner can read/write; `---` group has nothing; `---` others have nothing.
3. `/etc` = system configuration files; `/home` = users' home directories.

## Part 2. Processes
Commands: `ps aux | head`, `ps aux | wc -l`, `top`, `ps -p 1 -o pid,comm`, `sleep 300 &`, `jobs`, `ps -ef | grep sleep`, `kill`, `/proc/$PID/status`.
Interesting output:
```
ps aux | wc -l  -> 215
Tasks: 213 total, 1 running, 212 sleeping, 0 stopped, 0 zombie
PID 1 = systemd
```
(Note: `kill <PID>` and `kill $PID` failed at first because `<PID>` was a placeholder and `$PID` was not set yet.)
1. PID 1 is **systemd**, the init process started first by the kernel; it starts and supervises the other services.
2. About **213** processes, almost all sleeping (only 1 running).
3. `State: S (sleeping)`: waiting for its timer, not using the CPU.

## Part 3. Memory
Commands: `free -h`, `cat /proc/meminfo | head -6`, `grep VmRSS /proc/$PID/status`.
Interesting output:
```
Mem:  3.3Gi total  2.0Gi used  452Mi free  1.1Gi buff/cache  1.3Gi available
Swap: 3.8Gi total  620Ki used
VmRSS: 8360 kB
```
1. Total RAM about **3.3 GiB**; **452 MiB** free, **1.3 GiB** available (page cache can be given back to programs).
2. Swap = disk space used as overflow for RAM. **3.8 GiB** configured (file /swap.img), only 620 KiB used.
3. `sleep` uses **8360 kB (~8 MB)** resident memory. Somewhat surprising for a program that does nothing; mostly shared library pages mapped into memory.

## Part 4. Devices and storage
Commands: `df -h`, `lsblk`, `du -sh ~/os-lab1`, `ls -l /dev | head`, `mount | head`.
Interesting output:
```
/dev/sda2   24G  11G  12G  50% /
/dev/sda2 on / type ext4 (rw,relatime)
lrwxrwxrwx 1 root root 3 Oct 3 19:47 cdrom -> sr0
```
1. Root `/` is on **/dev/sda2** (ext4, 24 GB).
2. `/dev/sda` = the virtual 25 GB hard disk; `/dev/sr0` (= /dev/cdrom) = the virtual CD/DVD drive.
3. "Everything is a file": disks, terminals, CD drives and process info get a name in the file tree and are used with the same file operations (ls, cat, read, write).

## Bonus: `strace -c ls`
`ls` made **139 system calls** (most: mmap 22, mprotect 13, read 13, openat 11); `getdents64` (2 calls) reads the directory, `write` (1 call) prints the result.

## Conclusion
The OS manages four resources: files, processes, memory and devices. I saw files with `ls -l` and processes with `ps aux`/`top` (PID 1 is systemd, 213 processes). I saw memory with `free -h` (3.3 GiB RAM, 3.8 GiB swap) and devices with `df -h`/`lsblk` (root on /dev/sda2).
