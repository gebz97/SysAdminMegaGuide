#memory #mem #memory_management #free-h #vmstat

## Memory Architecture

```bash
cat /proc/meminfo
```

Ground truth: MemTotal, MemFree, MemAvailable (≠ MemFree), Buffers, Cached, SwapCached, Dirty, Writeback, HugePages_*.

```bash
free -h
```

Human-readable summary. `available` column is what actually matters — not `free`.

```bash
vmstat -s
```

Static memory stats dump. Good for scripting.

---

## Virtual Memory Behavior

```bash
cat /proc/sys/vm/swappiness
```

0–100. How aggressively kernel swaps. Desktop: 10. Server: 1–10. Never 0 on systems with swap (OOM risk).

```bash
sysctl -w vm.swappiness=10
```

Set live. Persist in `/etc/sysctl.conf`.

```bash
cat /proc/sys/vm/dirty_ratio
cat /proc/sys/vm/dirty_background_ratio
```

Max % of RAM that can be dirty pages before blocking writes / background writeback kicks in.

```bash
cat /proc/sys/vm/overcommit_memory
```

0=heuristic, 1=always allow, 2=never overcommit beyond swap+RAM*ratio. Production DBs often use 2.

```bash
cat /proc/sys/vm/overcommit_ratio
```

Used when overcommit_memory=2. Percentage of RAM committable beyond swap.

---

## Per-Process Memory

```bash
cat /proc/<PID>/status | grep -i vm
```

VmPeak, VmRSS, VmSwap, VmData, VmStk, VmExe, VmLib at a glance.

```bash
cat /proc/<PID>/smaps_rollup
```

Aggregated: Rss, Pss, Private_Clean, Private_Dirty, Shared_Clean, Shared_Dirty, Swap. The real numbers.

```bash
cat /proc/<PID>/smaps
```

Per-mapping breakdown. Every mmap region with full accounting. Slow but complete.

```bash
pmap -x <PID>
```

Readable smaps. RSS + dirty per mapping.

```bash
pmap -XX <PID>
```

Maximum detail — kernel-exposed columns, varies by version.

### Understanding the columns:

- **VSZ/VmSize** — virtual address space reserved. Meaningless for actual usage.
- **RSS** — pages currently in RAM. Double-counts shared libs.
- **PSS** — RSS but shared pages divided by # of processes sharing them. The honest number.
- **USS** — private pages only. What you'd free by killing this process.

---

## System-Wide Memory Pressure

```bash
vmstat 1
```

Live: `si`/`so` = swap in/out per second. Any nonzero `so` under load = memory pressure.

```bash
sar -r 1
```

Per-second memory stats via sysstat. `%memused`, `kbavail`, `kbdirty`.

```bash
sar -B 1
```

Paging: `pgscank` (pages scanned by kswapd), `pgsteal` (pages reclaimed). High scan rate = pressure.

```bash
sar -W 1
```

Swap in/out rates.

```bash
cat /proc/vmstat
```

Raw kernel counters: `pgscan_kswapd`, `pgsteal_kswapd`, `pgmajfault`, `oom_kill`. Delta these over time.

```bash
watch -n1 'grep -E "MemAvailable|Dirty|Writeback|SwapFree|Mlocked" /proc/meminfo'
```

Live watch on the fields that actually indicate trouble.

---

## OOM Killer

```bash
dmesg | grep -i 'oom\|killed process\|out of memory'
```

OOM kill history with victim, score, and memory state at time of kill.

```bash
grep -i oom /var/log/kern.log
```

Same from persistent log.

```bash
cat /proc/<PID>/oom_score
```

Current OOM score (0–1000). Higher = more likely to be killed.

```bash
cat /proc/<PID>/oom_score_adj
```

Adjustment (-1000 to 1000). -1000 = never kill. 1000 = kill first.

```bash
echo -500 > /proc/<PID>/oom_score_adj
```

Protect a critical process. Root only.

```bash
echo 1000 > /proc/<PID>/oom_score_adj
```

Sacrifice this process first under OOM.

```bash
cat /proc/sys/vm/panic_on_oom
```

0=OOM killer runs, 1=kernel panic instead. Some HA setups prefer panic+reboot over silent kills.

---

## Huge Pages

```bash
grep -i huge /proc/meminfo
```

HugePages_Total, Free, Rsvd, Surp, Hugepagesize, AnonHugePages (THP).

```bash
cat /proc/sys/vm/nr_hugepages
```

Static huge pages pre-allocated. Used by Oracle DB, DPDK, KVM guests.

```bash
sysctl -w vm.nr_hugepages=512
```

Pre-allocate 512 huge pages (2MB each = 1GB). Do this at boot — fragmentation prevents it later.

```bash
cat /sys/kernel/mm/transparent_hugepage/enabled
```

THP setting: `always`, `madvise`, `never`.

```bash
echo madvise > /sys/kernel/mm/transparent_hugepage/enabled
```

Only use THP for processes that explicitly request it. Recommended for most production systems — `always` causes latency spikes in Redis, Java, etc.

```bash
cat /sys/kernel/mm/transparent_hugepage/defrag
```

How aggressively kernel compacts memory to form huge pages. `defer+madvise` is usually sane.

---

## Memory Mapping & Locking

```bash
cat /proc/<PID>/maps
```

Every virtual memory region: address range, permissions (rwxp), offset, device, inode, name.

```bash
cat /proc/<PID>/numa_maps
```

Which NUMA node each mapping is on, how many pages are local vs remote.

```bash
cat /proc/sys/vm/max_map_count
```

Max VMAs per process. Java/Elasticsearch hit this. Default 65530. Raise if you see `mmap failed: ENOMEM`.

```bash
sysctl -w vm.max_map_count=262144
```

Standard fix for Elasticsearch.

```bash
mlockall(MCL_CURRENT|MCL_FUTURE)
```

Syscall to pin all process memory (no swapping). Done by Redis, real-time apps. Needs `RLIMIT_MEMLOCK` or CAP_IPC_LOCK.

```bash
cat /proc/<PID>/status | grep VmLck
```

How much memory the process has locked.

---

## NUMA

```bash
numactl --hardware
```

NUMA topology: nodes, CPUs per node, memory per node, distances.

```bash
numastat
```

Per-node: numa_hit, numa_miss, interleave_hit. High `numa_miss` = wrong-node allocations = latency.

```bash
numastat -p <PID>
```

Per-process NUMA allocation breakdown.

```bash
numactl --membind=0 --cpunodebind=0 ./app
```

Force process to allocate memory and run on NUMA node 0 only. Critical for latency-sensitive apps.

```bash
numactl --interleave=all ./app
```

Round-robin allocation across nodes. Better throughput for large in-memory datasets.

```bash
cat /proc/sys/kernel/numa_balancing
```

Auto-NUMA: kernel migrates pages toward the CPU using them. Can cause noise. Disable for latency-critical workloads.

---

## Page Cache & Reclaim

```bash
cat /proc/sys/vm/vfs_cache_pressure
```

100=default. Lower = kernel keeps dentries/inodes longer. Higher = more aggressive reclaim. Tune for workload.

```bash
sync && echo 1 > /proc/sys/vm/drop_caches
```

Drop page cache. `echo 2` drops dentries/inodes. `echo 3` drops all. **Never on production under load.**

```bash
fincore <file>
```

Which pages of a file are currently in page cache.

```bash
vmtouch -v <file>
```

Page cache residency per file. `-l` locks it in, `-e` evicts it.

```bash
cat /proc/sys/vm/min_free_kbytes
```

Reserve this much RAM — kswapd tries to keep at least this free. Raise on high-memory servers to avoid OOM storms.

---

## cgroup Memory Control (v2)

```bash
cat /sys/fs/cgroup/<slice>/memory.current
```

Current memory usage of a cgroup in bytes.

```bash
cat /sys/fs/cgroup/<slice>/memory.max
```

Hard limit. Processes hitting this get OOM-killed within the cgroup.

```bash
cat /sys/fs/cgroup/<slice>/memory.high
```

Soft throttle threshold. Kernel reclaims aggressively before hard limit is hit.

```bash
cat /sys/fs/cgroup/<slice>/memory.stat
```

Full breakdown: anon, file, slab, sock, pgfault, pgmajfault, oom_kill. The cgroup equivalent of /proc/meminfo.

```bash
cat /sys/fs/cgroup/<slice>/memory.pressure
```

PSI (Pressure Stall Information) for this cgroup. `some` and `full` stall percentages over 10s/60s/300s.

```bash
cat /proc/pressure/memory
```

System-wide PSI. `full` line nonzero = processes were completely stalled waiting for memory. Definitive pressure signal.

---

## Memory Leak Detection

```bash
valgrind --leak-check=full --track-origins=yes ./app
```

Full leak report with allocation origin. Slow — dev/staging only.

```bash
valgrind --tool=massif ./app
ms_print massif.out.<PID>
```

Heap profiler. Shows allocation growth over time with call stacks.

```bash
heaptrack ./app
heaptrack_print heaptrack.<app>.<PID>.zst
```

Faster than valgrind. Flame graph of heap allocations.

```bash
bpftrace -e '
  uprobe:/lib/x86_64-linux-gnu/libc.so.6:malloc { @bytes[pid] = sum(arg0); }
  uprobe:/lib/x86_64-linux-gnu/libc.so.6:free  { @bytes[pid] = sum(-1); }
  interval:s:5 { print(@bytes); }
'
```

eBPF live malloc/free delta per PID. Zero-instrumentation leak tracker.

```bash
watch -n2 'cat /proc/<PID>/status | grep VmRSS'
```

Blunt but instant — confirm RSS growing unbounded.

---

## Swap

```bash
swapon --show
```

All swap devices/files: size, used, priority.

```bash
cat /proc/swaps
```

Same, raw.

```bash
swapoff -a && swapon -a
```

Cycle swap — forces swapped pages back to RAM (if room). Useful post-memory-pressure cleanup.

```bash
smem -s swap -r | head -20
```

Top processes by swap usage.

```bash
for f in /proc/*/status; do
  awk -v f=$f '/VmSwap/{if($2>0) print f, $2, "kB"}' $f
done | sort -k2 -rn | head -20
```

Find swap consumers without smem.

---

## Profiling & Flamegraphs

```bash
perf record -g -e page-faults -p <PID> sleep 10
perf report
```

Which code paths are causing page faults.

```bash
perf stat -e page-faults,major-faults,cache-misses,cache-references -p <PID>
```

Memory-related hardware counters for a process.

```bash
bpftrace -e 'software:page-faults:1 /pid == <PID>/ { @[ustack] = count(); }'
```

Page fault flame data grouped by userspace call stack. Pipe to FlameGraph tools.

```bash
perf mem record -p <PID> sleep 10
perf mem report
```

Memory access latency profiling — shows load/store latency broken down by cache level.