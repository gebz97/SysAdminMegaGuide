#linux #process_management #sysadmin
#ps
## Process Snapshot

```bash
ps aux
```
BSD style. Key columns: `%CPU`, `%MEM`, `VSZ`, `RSS`, `STAT` (`R`=running, `S`=sleeping, `D`=blocked, `Z`=zombie).

```bash
ps -efH
```
Solaris style with hierarchy. Shows `PPID` (parent) and tree structure.

---
## Real-Time Monitoring

```bash
top -d 1
```
1-second refresh. Press `P` to sort by CPU, `M` by memory, `k` to kill a PID, `r` to renice.

```bash
htop
```
Color, mouse, scrollable. `F5` for tree view.

---
## Find PIDs

```bash
pgrep -l nginx
```
Match by exact process name, list with name.

```bash
pgrep -u www-data
```
All processes owned by a user.

```bash
pgrep -f "python app.py"
```
Match against full command line (for scripts, args).

---
## Process Tree

```bash
pstree -p <PID>
```
Tree of parent/child for a specific PID. `-a` shows arguments, `-u` shows UID transitions.

---
## Signals & Killing

```bash
kill -15 <PID>
```
SIGTERM: polite shutdown, always try first.

```bash
kill -9 <PID>
```
SIGKILL: nuclear option, kernel kills it instantly, no cleanup.

```bash
kill -1 <PID>
```
SIGHUP: reload config without restarting the process.

```bash
killall -15 nginx
```
By name.

```bash
pkill -U www-data
```
By user.

```bash
pkill -f "python server.py"
```
Matches full command line.

---
## Pause/Resume

```bash
kill -STOP <PID>
```
Freeze a process (SIGSTOP, cannot be caught).

```bash
kill -CONT <PID>
```
Unfreeze it.

---
## Foreground/Background (Shell Jobs)

```bash
Ctrl+Z
```
Suspend the foreground process.

```bash
bg
```
Resume the suspended job in the background.

```bash
fg
```
Bring background job to the foreground.

```bash
jobs -l
```
List shell jobs with PIDs.

```bash
nohup ./long_task.sh &
```
Start in background, immune to hangup (survives terminal close).

---
## Priority

```bash
nice -n 10 ./task.sh
```
Launch with lower priority (`-20` highest, `19` lowest, default `0`).

```bash
renice -n -5 -p <PID>
```
Change running process priority. Need root for negative values (higher priority).

---
## CPU Pinning

```bash
taskset -c 0,1 ./task.sh
```
Lock process to CPU cores 0 and 1.

```bash
taskset -cp 0 <PID>
```
Move a running process to core 0 only.

---
## Inspect a Process via /proc

```bash
ls -l /proc/<PID>/fd/
```
See all open file descriptors (files, sockets, pipes). Count for leak detection.

```bash
cat /proc/<PID>/status
```
Quick view: process state, threads, memory, UID/GID.

```bash
cat /proc/<PID>/limits
```
Current ulimit values for that process.

```bash
cat /proc/<PID>/cmdline | tr '\0' ' '
```
Full command line with arguments.

```bash
ls -l /proc/<PID>/cwd
```
Current working directory of the process.

---
## Deep Diagnostics

```bash
strace -p <PID>
```
Trace all system calls in real time. See what the process is actually doing (hanging? permission errors?).

```bash
strace -e trace=open,close,read,write -p <PID>
```
Filter to file and socket I/O calls only.

```bash
strace -c -p <PID>
```
Attach, let it run, Ctrl+C — prints time-spent summary per syscall. Find bottlenecks instantly.

```bash
lsof -p <PID>
```
All files, sockets, pipes opened by process.

```bash
lsof -i :8080
```
Which process is listening on port 8080.

```bash
fuser -v 22/tcp
```
PID holding a TCP port.

```bash
pidstat -p <PID> 1
```
Per-second CPU usage for a PID.

```bash
pidstat -d -p <PID> 1
```
Per-second disk I/O for a PID.

```bash
perf top -p <PID>
```
Live CPU profile of what functions inside the process are burning cycles.

---
## Namespace & Cgroup Inspection

```bash
ls -l /proc/<PID>/ns/
```

Show all namespaces (net, pid, mnt, uts, ipc, user) a process belongs to.

```bash
nsenter -t <PID> --net --pid bash
```

Enter a process's namespaces (essential for container debugging without `docker exec`).

```bash
cat /proc/<PID>/cgroup
```

Which cgroup(s) the process is in.

```bash
systemd-cgls
```

Full cgroup tree with processes.

```bash
systemctl status <PID>
```

If systemd-managed: unit, cgroup, recent logs, resource limits — all at once.

---
## Memory Deep Dive

```bash
cat /proc/<PID>/smaps
```

Per-mapping breakdown: PSS, private/shared dirty, swap usage. The real memory cost of a process.

```bash
cat /proc/<PID>/smaps_rollup
```

Aggregated smaps — faster, one-liner for total PSS/swap.

```bash
pmap -x <PID>
```

Readable smaps alternative. `-X` shows even more columns (kernel ≥3.x).

```bash
cat /proc/<PID>/oom_score
cat /proc/<PID>/oom_score_adj
```

OOM killer score. Adjust with `oom_score_adj` (-1000 to 1000) to protect or sacrifice a process.

```bash
echo -500 > /proc/<PID>/oom_score_adj
```

Make OOM killer less likely to target this process.

---

## File Descriptor Limits & Leak Hunting

```bash
ls /proc/<PID>/fd | wc -l
```

Quick FD count. Compare against `/proc/<PID>/limits` `Max open files`.

```bash
watch -n1 'ls /proc/<PID>/fd | wc -l'
```

Live FD growth — confirms a leak in seconds.

```bash
inotifywait -m /proc/<PID>/fd
```

Get notified on every FD open/close.

---

## Scheduler & Latency

```bash
chrt -p <PID>
```

Show scheduling policy (SCHED_OTHER, SCHED_FIFO, SCHED_RR, SCHED_BATCH, SCHED_IDLE).

```bash
chrt -f -p 50 <PID>
```

Set SCHED_FIFO real-time priority 50. Needs root. Use carefully.

```bash
chrt -b -p 0 <PID>
```

Set SCHED_BATCH — good for CPU-heavy background jobs.

```bash
cat /proc/<PID>/schedstat
```

Raw: time on CPU, time waiting on runqueue, # of timeslices. Useful for latency profiling.

```bash
/proc/<PID>/sched
```

Verbose scheduler stats if `CONFIG_SCHED_DEBUG` is enabled.

---

## Tracing Beyond strace

```bash
ltrace -p <PID>
```

Library call trace (libc, etc.) — complements strace for userspace.

```bash
perf record -g -p <PID> sleep 10
perf report
```

Flame-graph-ready CPU profile. `-g` captures call graphs.

```bash
perf stat -p <PID>
```

Hardware counters: cache misses, branch mispredictions, IPC. Instant CPU efficiency read.

```bash
bpftrace -e 'tracepoint:syscalls:sys_enter_* /pid == <PID>/ { @[probe] = count(); }'
```

eBPF: count every syscall by type for a PID, zero overhead approach.

```bash
bpftrace -e 'kprobe:do_sys_open /pid == <PID>/ { printf("%s\n", str(arg1)); }'
```

Trace every file open by PID at kernel level — catches what strace misses under ptrace restrictions.

---

## Signal Masking

```bash
grep -E 'Sig(Blk|Ign|Cgt)' /proc/<PID>/status
```

Bitmask of blocked/ignored/caught signals. Decode with:

```bash
python3 -c "print(bin(int('0000000000004000', 16)))"
```

Or use `kill -l` to map bit positions to signal names. Critical when a process silently swallows signals.

---

## Process Accounting & Audit

```bash
lastcomm <process_name>
```

Historical execution log (requires `acct` service running). Who ran it, when, how long, exit code.

```bash
ausearch -p <PID>
```

All auditd events for a PID — exec, file access, network, privilege escalation.

```bash
auditctl -a exit,always -F pid=<PID> -S all
```

Live audit rule: log every syscall for this PID going forward.

---

## Inter-Process & Socket Internals

```bash
ss -tp | grep <PID>
```

All TCP sockets with owning PID.

```bash
ss -xp | grep <PID>
```

Unix domain sockets.

```bash
cat /proc/<PID>/net/tcp
```

Raw hex socket table for the process's net namespace. Decode local/remote addr from hex.

```bash
netstat -anp | grep <PID>
```

Legacy but still reliable on older systems.

---

## Zombie Reaping

```bash
ps aux | awk '$8 == "Z"'
```

Find all zombies.

```bash
cat /proc/<zombie_PID>/status | grep PPid
```

Get parent. The parent must `wait()` — you can't kill a zombie, only its parent.

```bash
kill -CHLD <PPID>
```

Signal parent to reap. If parent is stuck or ignoring SIGCHLD, kill the parent.

---

## Core Dumps

```bash
cat /proc/sys/kernel/core_pattern
```

Where/how cores are written. May pipe to `systemd-coredump` or `apport`.

```bash
ulimit -c unlimited
```

Enable core dumps for current shell session.

```bash
gcore <PID>
```

Dump core of a **running** process without killing it. Invaluable.

```bash
gdb <binary> core.<PID>
bt full
```

Full backtrace from a core dump.

---

## Execution & Binary Inspection

```bash
ls -l /proc/<PID>/exe
```

Actual binary on disk (even if deleted — shows `(deleted)`).

```bash
cp /proc/<PID>/exe /tmp/recovered_binary
```

Recover a deleted binary from a running process.

```bash
cat /proc/<PID>/maps
```

Full memory map: every loaded library, heap, stack, anonymous mappings with permissions.

---

## Systemd-Specific

```bash
systemctl show <service> --property=MainPID,MemoryCurrent,CPUUsageNSec
```

Pull specific resource properties programmatically.

```bash
journalctl _PID=<PID>
```

All logs ever emitted by that exact PID.

```bash
systemd-run --scope -p MemoryMax=512M ./task.sh
```

Run a process inside a transient cgroup with hard memory cap — no unit file needed.