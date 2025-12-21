Below is a **single, structured study document** you can revise end-to-end for a Google TSE (Compute / Linux internals) round. It’s designed to eliminate repeat questions by giving you:

* **Simple explanation (no jargon)**
* **Clarifications & common traps**
* **What Google expects you to say**
* **Tools + exact commands + key flags**
* **Sample outputs + how to read them**
* **Fix / remediation patterns** (Google cares a lot here)
* **10 mock scenarios with “expected answer scripts”**
* **MMU & page tables deep dive (ASCII)**
* **One-page printable command → interpretation sheet**

---

# Linux Internals & OS Study Guide (Google TSE)

## How to use this document

When answering in interview, follow this consistent shape:

1. **Classify the problem**: CPU vs Memory vs Disk/IO vs Network vs App logic
2. **Prove with 1–2 tools** (not 10 commands)
3. **Explain the likely root cause**
4. **Propose a fix + prevention** (rate limiting, sharding, async, better limits, observability)

This “prove → explain → fix” pattern is the strongest signal.

---

# 1) Linux Under the Hood

## 1.1 Kernel

### What it is (simple)

The **kernel** is the core OS program that controls:

* CPU scheduling
* memory mapping
* processes/threads
* filesystem and disks
* networking stack
* device drivers

User programs can’t touch hardware directly; they must ask the kernel.

### Google expects you to say

“User space requests privileged operations via system calls; kernel enforces isolation and scheduling.”

### Tools & commands

```bash
uname -a
uname -r
dmesg | tail -n 50
journalctl -k -n 50 --no-pager
```

### When to use

* boot failures, driver issues, kernel panic evidence, OOM killer, hardware-level errors.

---

## 1.2 Shell (what it really does)

### Concept (simple)

The shell:

1. parses text into tokens
2. expands variables (`$HOME`), globs (`*.log`), command substitution (`$(...)`)
3. finds binary using `$PATH`
4. launches process with `execve()`

### Trap clarifications

* **Globbing happens before program runs.**
* Quotes stop expansion: `"*.log"` is literal, not expanded.

### Tools

```bash
echo $PATH
type ls
which ls
```

---

## 1.3 Libraries, system calls, and “libcall vs syscall”

### The core meaning

* **Library call (libcall)**: normal user-space function call (e.g., `printf`, `malloc`)
* **System call (syscall)**: a controlled entry into kernel mode (`write`, `read`, `openat`, `futex`)

### Why `printf()` isn’t a syscall

`printf()` formats text and often buffers it; it *may* later call `write()` syscall when flushing.

**Typical chain**

```
printf("hi\n")
  -> format in user-space
  -> (flush happens)
  -> write(1, "hi\n", 3)  <-- syscall boundary
```

### Prove it with `strace`

`strace` shows **syscalls** only.

```bash
strace -e write ./a.out
```

Sample:

```
write(1, "hi\n", 3) = 3
```

### Google trap

“Is `printf` a syscall?” → **No**.
“Why didn’t output appear immediately?” → **stdout buffering**.

---

## 1.4 Regex vs globbing (the “* behaves differently” trap)

### Simple rule

* **Globbing**: shell filename matching (`*.log`)
* **Regex**: pattern language used by `grep/sed/awk`

### What `*` means

* glob `*` = “any characters in filenames”
* regex `*` = “repeat the previous token 0+ times”

### Examples

Globbing:

```bash
ls *.log
```

Regex:

```bash
grep -E ".*\.log" file.txt
```

### Trap

`grep "*.log"` doesn’t do what you think:

* quotes stop glob expansion
* regex `*` repeats the previous character/token

---

# 2) Processes, Threads, and Resources

## 2.1 Process

### Concept

A **process** is a running program with:

* its own **virtual address space**
* its own **PID**
* its own **credentials** (uid/gid/caps)
* its own **file descriptor table**
* one or more threads

### Resources a process needs (interview-ready)

* CPU time (scheduled)
* memory mappings (address space)
* file descriptors (files, sockets, pipes)
* credentials/permissions
* kernel objects (fds, locks, signals, timers)

### Key OS mechanics

* `fork()` creates child (copy-on-write memory)
* `execve()` replaces process image with a new program
* scheduler runs runnable tasks

### Tools

```bash
ps -eo pid,ppid,stat,ni,cmd --sort=-%cpu | head
lsof -p <PID> | head
cat /proc/<PID>/status | head -n 30
```

---

## 2.2 Thread

### Concept

Threads share:

* address space (heap/globals)
* file descriptors
  But each thread has its own:
* registers
* stack
* scheduling identity (TID)

### Why threads can be faster (and why they can be worse)

* Fast communication via shared memory
* BUT: shared memory needs coordination → locks → contention → context switches

### Tools

```bash
top -H -p <PID>
ps -L -p <PID> -o pid,tid,psr,stat,pcpu,comm | head
```

---

# 3) CPU Scheduling, Context Switching, and Concurrency

## 3.1 CPU Scheduling (CFS)

### Simple model

Linux’s **CFS** tries to give tasks a fair share. It tracks **virtual runtime**:

* tasks with less CPU usage (smaller vruntime) run sooner
* tasks are organized (conceptually) in a balanced tree; scheduler picks the “fairest next”

### Priorities in practice

* `nice`/`renice` adjust weight under CFS (not “real-time”)
* realtime policies (SCHED_FIFO/RR) are a separate scheduling class

### Tools

```bash
ps -eo pid,ni,cls,rtprio,comm | head
renice -n -5 -p <PID>
chrt -p <PID>
```

### Google traps

* “nice makes it realtime” → **No**
* “why a nice’d process still uses CPU” → fairness is relative, not absolute.

---

## 3.2 Context Switching (deep but simple)

### What it is

A **context switch** is when CPU stops running task A and starts task B:

* save A registers/PC/SP
* load B state
* if switching processes: switch page tables (address space)
* costs time + disrupts caches

### What triggers it (your bullet list, clarified)

#### (1) Timer interrupt (preemption)

Kernel regains control periodically and may schedule another runnable task.

#### (2) Blocking syscall (voluntary switch)

If a task calls something that can’t complete immediately (disk, socket, lock), it sleeps; scheduler runs someone else.

#### (3) Higher-priority task wakes up

A higher-priority runnable task may preempt the current one.

---

## 3.3 High context switches (cs): what it really means

### Where you see it

`vmstat 1` → `cs` = context switches per second.

### Two main causes (most interview-relevant)

#### Cause A: Lock contention (many wait/wake cycles)

Threads fight over locks:

* block on lock → sleep
* lock released → wake another → switch
* repeated rapidly = high `cs`, often high `sy`

Evidence:

* `strace -e futex` shows FUTEX_WAIT / FUTEX_WAKE storms
* perf shows time in locking/scheduler paths

#### Cause B: Oversubscription (too many runnable threads)

More runnable threads than CPU cores:

* scheduler time-slices constantly
* high `cs`
* run queue (`r`) stays high

Evidence:

* `vmstat` shows `r` consistently > CPU cores
* `top -H` shows many runnable threads

### How to differentiate quickly (Google-grade)

1. `vmstat 1`: check `r`, `cs`, `us/sy/wa/st`
2. contention proof: `strace -f -e futex -p PID`
3. oversubscription proof: thread count + `r` high

---

# 4) Locks & Modern Concurrency Constructs (must-know)

## 4.1 Why locks exist

Shared memory causes races:

* corrupted structures
* lost increments
* inconsistent reads

Locks serialize access to critical sections.

---

## 4.2 Mutex

### Meaning

A **mutex** allows **one owner** at a time (mutual exclusion).

### Typical symptoms when it’s bad

* many threads wait on same mutex → poor scaling, high context switches
* deadlock if acquisition order inconsistent or unlock forgotten

### Linux evidence

* futex waits (pthread mutex uses futex under contention)

---

## 4.3 Semaphore

### Meaning

A **semaphore** allows **N permits** concurrently (counting semaphore).

Use cases:

* limit DB connections to 100
* bound concurrency of a resource

### Interview clarification

Mutex = exclusive ownership
Semaphore = capacity control (not necessarily “owned” the same way)

---

## 4.4 Monitor

### Meaning

A **monitor** = mutex + condition variables.
It solves “wait until condition becomes true” safely.

Producer/consumer:

* lock
* if queue empty → wait (releases lock + sleeps)
* producer pushes item → signal → consumer wakes

---

## 4.5 Futex (Fast Userspace Mutex)

### Why it exists

Locks should be fast when uncontended:

* uncontended path uses atomics in user space
* only contended waits/wakes enter kernel via futex syscall

### What you see in debugging

```bash
strace -f -e futex -p <PID>
```

Sample patterns:

```
futex(0x..., FUTEX_WAIT, ...) = 0
futex(0x..., FUTEX_WAKE, ...) = 1
```

---

## 4.6 Fixes Google expects for contention

These are not buzzwords—each maps to a real structural change.

### Reduce shared state

* make data immutable where possible
* use thread-local state
* single-writer / actor approach

### Shard locks

Split one hot lock into many:

* hash buckets each with a lock
* per-CPU/per-shard counters

### Async design

Replace “one thread per request” with:

* non-blocking IO + event loop
* bounded worker pool for CPU-heavy work

Also mention:

* shrink critical sections
* avoid locking around IO
* use read-write locks if reads dominate (with caution)

---

# 5) Deadlock & Livelock

## 5.1 Deadlock

Circular dependency where no one can proceed.

### “Break one condition” (your bullet, fully explained)

Deadlock requires 4 conditions. Prevent by ensuring at least one cannot happen:

#### Ordering (break circular wait) — best answer

Define a global lock order, always acquire in that order.

#### Timeout / try-lock (reduce infinite wait)

If you can’t acquire lock B, release lock A and retry later.

#### Preemption / rollback (break no-preemption)

Abort one task/transaction and release locks.

### Google expects

You name **ordering first**, then timeout/try-lock as pragmatic.

---

## 5.2 Livelock

Threads are running and “doing work” but making no progress (e.g., both keep backing off and retrying).

Fix:

* randomness/jitter in retry
* backoff strategy
* redesign to avoid symmetric conflict

---

# 6) Perf, IPC, and CPU Diagnosis

## 6.1 perf basics

### What perf is

A profiler using hardware counters and sampling to see where CPU time goes.

### Commands

```bash
perf stat -p <PID> sleep 5
perf top -p <PID>
```

---

## 6.2 IPC (instructions per cycle)

### Meaning

IPC tells you how efficiently CPU cycles are used.

**Low IPC means CPU is often waiting**, not executing instructions.

### Causes you must connect (your bullet)

* stalls due to memory/cache misses
* branch mispredicts
* lock contention / kernel scheduling overhead

### How to tie it to other signals

If IPC low:

* check cache miss/stalled cycles in `perf stat`
* check `vmstat cs` + futex (`strace`) for lock contention
* check IO wait (`iostat`, `vmstat wa`) if it’s blocking

---

# 7) Memory Management (including ballooning)

## 7.1 Memory pressure

### Signals

```bash
free -m
vmstat 1
cat /proc/meminfo | head
```

* low MemAvailable
* rising swap in/out (`si/so`) = pressure

### OOM killer proof

```bash
dmesg | grep -i -E "oom|killed process" | tail
journalctl -k | grep -i oom | tail
```

---

## 7.2 Memory ballooning (your bullet, clarified)

### What it is

In VMs, hypervisor may reclaim guest memory using a “balloon driver”:

* guest pins memory
* host reclaims it
* guest sees less available RAM, may swap more

### Symptoms

* sudden drop in available memory
* increased swap activity
* latency spikes without app RSS growth

### Tools

```bash
free -m
vmstat 1
dmesg | tail -n 50
```

### Google expectation

Consider external contention (“noisy neighbor”, host-level reclaim) as a possibility.

---

# 8) Disk / Filesystems / Inodes / Links

## 8.1 Disk IO wait

### Tools

```bash
iostat -x 1
vmstat 1
pidstat -d 1
```

Interpretation:

* high `%util` + high `await` → saturated disk
* high `wa` in vmstat → CPU waiting on IO

---

## 8.2 Disk full (classic trap: deleted but open)

### Tools

```bash
df -h
du -sh * | sort -h
lsof +L1
```

If `df` says full but `du` doesn’t show it:

* `lsof +L1` often reveals huge deleted files still open by processes.

---

## 8.3 Inodes

### Concept

Inode = file metadata (owner, perms, timestamps, pointers to blocks).
Filename is a directory entry mapping name → inode.

### Tools

```bash
df -i
ls -li
```

---

## 8.4 Hard links vs symlinks

### Hard link

Two names → same inode (same file data)

* deleting one name doesn’t delete data until link count reaches 0

### Symlink

A separate file containing a path pointer.

Tools:

```bash
ls -li
stat file
```

---

# 9) Boot Process (detailed, with troubleshooting)

## 9.1 Stages

```
BIOS/UEFI
  -> Bootloader (GRUB)
    -> Kernel + initramfs
      -> init/systemd
        -> services
```

## 9.2 What breaks where (and what you check)

### BIOS/UEFI

* no boot device / wrong order

### GRUB

* grub rescue prompt, wrong root UUID

### Kernel stage

* kernel panic, missing driver, initramfs issues

### systemd stage

* service dependency loops, failed mounts, wrong unit configs

## Tools

```bash
journalctl -xb --no-pager
journalctl -k --no-pager | tail -n 80
systemctl --failed
```

---

# 10) Advanced Debugging Toolkit (what to use when)

## 10.1 Tool selection table (memorize)

* **strace**: process stuck / unknown waits → syscall truth
* **perf**: high CPU / throughput drop → hot paths
* **tcpdump**: network handshake, retransmits, TLS evidence
* **dmesg / journalctl -k**: kernel logs, OOM, device errors
* **coredumpctl**: crashes/segfaults
* **lsof**: FD leaks, deleted-open files, port binds
* **ss**: sockets, listening ports, connection states

---

# 11) 10 Google-style Mock Scenarios (expected answer scripts)

Use these to practice speaking.

## 1) High CPU, low throughput

1. “Confirm CPU bound: `top`, `pidstat -u 1`.”
2. “Find hot path: `perf top -p PID`.”
3. “If lock-y: check `vmstat cs` + `strace -e futex`.”
4. “Fix: shard locks, cap threads, reduce shared state, async for IO workloads.”

## 2) High context switches + latency spikes

1. “`vmstat 1` → check `r` and `cs`.”
2. “If `r` high → oversubscription; reduce threads/pool size.”
3. “If futex storms → contention; shard locks + shrink critical section.”

## 3) Service stuck, low CPU

1. “Attach `strace -f -p PID`.”
2. “If `futex` wait → lock/possible deadlock.”
3. “If `read/poll` → waiting on IO; check upstream / sockets `ss`.”

## 4) IO wait high, CPU mostly idle

1. “`iostat -x 1` + `vmstat wa`.”
2. “Find offender: `pidstat -d 1`.”
3. “Fix: caching, reduce sync IO, faster storage, batching.”

## 5) Disk full but can’t find files

1. “`df -h` vs `du` mismatch.”
2. “Run `lsof +L1` for deleted-but-open.”
3. “Fix: restart process / log rotation.”

## 6) systemd service restart loop

1. “`systemctl status svc` for exit code.”
2. “`journalctl -u svc` for precise error.”
3. “Fix perms, env, dependencies, unit config.”

## 7) VM shows high steal time

1. “`mpstat` shows `%st` high.”
2. “Conclusion: host contention/noisy neighbor.”
3. “Fix: move host/scale/change instance type.”

## 8) Random p99 spikes under load

1. “Check contention: `vmstat cs`, `strace futex`.”
2. “Check memory pressure: `free`, `vmstat si/so`.”
3. “Check IO: `iostat`.”
4. “Fix: cap concurrency, shard locks, reduce allocations.”

## 9) App crashes (segfault)

1. “Check core dump: `coredumpctl list`.”
2. “Backtrace: `coredumpctl gdb`.”
3. “Fix: patch/rollback, add guardrails, enable better crash capture.”

## 10) Network issue: curl vs browser mismatch

1. “Separate DNS/TCP/TLS/HTTP.”
2. “Use `curl -v`, `ss`, `tcpdump`.”
3. “Browser-only failures: cookies, CORS, HSTS, proxies, SNI/Host differences.”

---

# 12) MMU & Page Tables Deep Dive (ASCII + traps)

## 12.1 What MMU does

MMU translates **Virtual Address (VA)** → **Physical Address (PA)** and enforces permissions:

* process isolation
* kernel/user separation

## 12.2 Virtual memory layout (typical)

```
High addresses
+------------------+  stack (grows down)
|       stack      |
+------------------+
| mmap / shared so |
+------------------+
|       heap       |  (grows up)
+------------------+
|      code        |
+------------------+
Low addresses
```

## 12.3 Page tables and translation flow

```
CPU uses VA
  |
  v
TLB lookup (VA->PA cache)
  | hit -> done
  | miss
  v
Page table walk:
  L1 -> L2 -> ... -> PTE (maps to physical frame + perms)
  |
  v
Permission check (R/W/X, user/kernel)
  |
  +-- ok -> memory access proceeds
  |
  +-- fail/missing -> page fault -> kernel handler
```

## 12.4 Page fault vs segfault (critical trap)

* **Page fault**: kernel gets a trap because mapping/perms not satisfied *yet*
  can be normal: demand paging, copy-on-write
* **Segfault**: kernel decides access is invalid and sends SIGSEGV to process

So: **not all page faults are bad**; segfault is the “fatal” outcome.

## 12.5 Copy-on-write (fork)

```
fork()
 parent pages: shared read-only with child
 write happens
 -> page fault
 -> kernel copies page
 -> writer gets private writable copy
```

## 12.6 Why this matters for performance

* TLB misses → page table walks → slower
* context switching + big working sets → poor cache/TLB locality
* memory pressure → swapping → huge latency

---

# 13) One-page printable command → interpretation sheet

## CPU / Scheduling

```bash
top
top -H -p <PID>
```

* high CPU process/thread; per-thread view matters

```bash
vmstat 1
```

* `r` > cores → oversubscription
* `cs` high → contention or oversubscription
* `sy` high → kernel overhead (syscalls/locks)
* `wa` high → IO wait
* `st` high → VM steal time

```bash
mpstat 1
```

* `%st` high → noisy neighbor / host contention

## Profiling / Contention

```bash
perf stat -p <PID> sleep 5
```

* low IPC → stalls (memory/branch/locks)
* confirm with futex/caches

```bash
perf top -p <PID>
```

* where CPU time goes (hot functions)

```bash
strace -f -p <PID>
strace -f -e futex -p <PID>
```

* stuck? see syscall
* futex storms? lock contention

## Memory

```bash
free -m
vmstat 1
```

* low available + swap activity → pressure

```bash
dmesg | grep -i oom | tail
journalctl -k | grep -i oom | tail
```

* OOM killer evidence

## Disk / IO

```bash
iostat -x 1
pidstat -d 1
```

* high await/util → disk bottleneck

```bash
df -h
du -sh * | sort -h
lsof +L1
```

* disk full root cause; deleted-open files

## Services / Boot

```bash
systemctl --failed
systemctl status <svc>
journalctl -u <svc> -n 50 --no-pager
journalctl -xb --no-pager | tail -n 80
```

## Network (Linux-side)

```bash
ss -lntp
ss -antp | head
tcpdump -i any port 443
curl -v https://host
```

---

# Final prep recommendations (what to drill hardest)

For Google TSE, the highest-signal areas are:

1. **Classify incidents quickly** (CPU vs IO vs memory vs network)
2. **Context switching + contention vs oversubscription** (vmstat + strace futex)
3. **perf interpretation** (IPC + “what would you do next?”)
4. **systemd + journald** (service failures)
5. **boot chain mental model**
6. **MMU/page faults vs segfaults** (traps)

---

Here’s the **practice pack**: **20 rapid-fire Google-style prompts** with **ideal 20–30 second answers**, plus **follow-up traps** interviewers often add. Use it like flashcards: read the prompt → speak the answer out loud → check the trap.

---

# 20 Rapid-Fire Prompts + Ideal Answers (Google TSE / Linux Internals)

## 1) “What’s the kernel, in one sentence?”

**Ideal answer (20–25s):**
“The kernel is the core OS component that manages CPU scheduling, memory, devices, filesystems, and networking. User programs run in user mode and request privileged operations through system calls; the kernel enforces isolation and resource control.”

**Trap follow-up:** “Can user-space access hardware directly?” → “No, not safely; it must go via kernel mechanisms.”

---

## 2) “What is a system call? Give an example.”

**Ideal answer:**
“A system call is the controlled interface from user space into the kernel to perform privileged work like file IO or networking. Examples include `read`, `write`, `openat`, `accept`, and `futex`. You can see syscalls with `strace`.”

**Trap:** “Is `printf` a syscall?” → “No, it’s a libc call that may call `write` later.”

---

## 3) “Explain libcall vs syscall: why `printf → write`?”

**Ideal answer:**
“`printf()` is a user-space libc function that formats output and often buffers it. When the buffer flushes, libc calls the `write()` syscall to hand data to the kernel. `strace -e write` shows `write()`, not the internal formatting.”

**Trap:** “Why doesn’t output appear immediately?” → “Buffering; stdout is line-buffered on terminal, fully buffered when redirected.”

---

## 4) “Regex vs globbing — what’s the difference?”

**Ideal answer:**
“Globbing is done by the shell to expand filenames like `*.log` before the program runs. Regex is pattern matching inside tools like `grep`. In globbing `*` means ‘any characters’, but in regex `*` repeats the previous token, so `.*` is the ‘anything’ pattern.”

**Trap:** “Why does `grep "*.log"` fail?” → “Quotes prevent glob expansion; regex `*` needs a preceding token.”

---

## 5) “What happens when you run a command in the shell?”

**Ideal answer:**
“The shell parses tokens, expands variables and globs, resolves the executable using `$PATH`, then calls `fork()` and `execve()` to run it. `execve()` replaces the process image with the program.”

**Trap:** “Why is `$PATH` important?” → “It controls where the shell searches executables.”

---

## 6) “Process vs thread — explain like you’re teaching.”

**Ideal answer:**
“A process has its own virtual address space and isolation. Threads share the same address space and file descriptors but have separate stacks and registers. Threads communicate faster but need synchronization, so heavy locking can remove the advantage.”

**Trap:** “Why do threads reduce fault containment?” → “One bad write can crash the whole process.”

---

## 7) “What resources does a process need?”

**Ideal answer:**
“CPU time, memory mappings/address space, file descriptors for IO, scheduling structures, and security credentials. In Linux you inspect it via `/proc/<pid>/status`, `lsof`, and `ps`.”

**Trap:** “What’s the difference in resources for a thread?” → “Threads share address space and FDs; each has its own stack/register context and TID.”

---

## 8) “What is context switching?”

**Ideal answer:**
“A context switch is when the OS stops one task and runs another by saving/restoring CPU registers and scheduling state; for processes it also changes the active address space via page tables. It costs time and hurts cache locality.”

**Trap:** “Why is it expensive?” → “Cache/TLB disruption + kernel overhead.”

---

## 9) “What triggers a context switch?”

**Ideal answer:**
“Three common triggers: timer interrupt (preemption), blocking syscalls like IO or lock waits (the task sleeps), and a higher-priority task becoming runnable which can preempt the current one.”

**Trap:** “Can a running task switch without a timer?” → “Yes, if it blocks or is preempted by priority.”

---

## 10) “`vmstat 1` shows high `cs`. What does that suggest?”

**Ideal answer:**
“High context switches often means either oversubscription—too many runnable threads for the CPU—or lock contention where threads repeatedly sleep/wake on locks. I check `r` vs core count and confirm contention with `strace -e futex` or `perf`.”

**Trap:** “What’s the quickest differentiator?” → “If `r` stays high → oversubscription. Futex storms → contention.”

---

## 11) “Explain oversubscription in one example.”

**Ideal answer:**
“If you run 200 runnable threads on a 4-core VM, the scheduler time-slices constantly, raising `cs` and latency while each thread makes little progress. Fix is to cap workers, use bounded thread pools, or async IO instead of thread-per-request.”

**Trap:** “Is more threads always faster?” → “No—after core count it often hurts.”

---

## 12) “Locks vs mutex vs semaphore vs monitor — explain quickly.”

**Ideal answer:**
“A mutex is exclusive ownership: one holder at a time. A semaphore allows up to N concurrent holders, useful to limit concurrency. A monitor combines a mutex with condition variables so threads can wait for a condition safely—classic producer/consumer.”

**Trap:** “What’s the difference between waiting and spinning?” → “Spinning burns CPU; waiting sleeps.”

---

## 13) “What is a futex and why does Linux use it?”

**Ideal answer:**
“A futex is a ‘fast userspace mutex’: uncontended lock/unlock happens in user space using atomics. Only when contended does the kernel step in to sleep/wake threads via the `futex()` syscall. If I see many `FUTEX_WAIT/Wake`, it’s usually contention.”

**Trap:** “How do you observe it?” → “`strace -f -e futex -p PID`.”

---

## 14) “Your service is slow due to lock contention — what fixes do you propose?”

**Ideal answer:**
“First I prove contention with futex waits and high `cs`. Fixes: reduce shared state, shorten critical sections, shard locks to avoid one hot lock, and limit concurrency. Often an async/event loop model reduces threads and contention for IO-bound workloads.”

**Trap:** “Give an example of sharding locks.” → “Hash table buckets each with its own lock.”

---

## 15) “What is deadlock and how do you prevent it?”

**Ideal answer:**
“Deadlock is circular waiting: each thread holds a resource and waits for another. Prevention: enforce consistent lock ordering (best), use try-lock with timeout and rollback, or make some resources preemptible/abortable.”

**Trap:** “Why ordering works?” → “It breaks the circular-wait condition.”

---

## 16) “Deadlock vs livelock?”

**Ideal answer:**
“In deadlock nothing progresses and threads are stuck waiting. In livelock threads are active—retrying/backing off—but still make no progress. Livelock often needs jitter/backoff changes or redesign to avoid symmetric retries.”

**Trap:** “How would livelock show in CPU?” → “Often higher CPU than deadlock.”

---

## 17) “What is IPC and why do you care?”

**Ideal answer:**
“IPC is instructions per cycle. Low IPC means the CPU is stalled—waiting on memory, branches, or synchronization. I use `perf stat` to see IPC and cache misses, then correlate with `vmstat cs` and futex activity to separate memory-bound vs lock-bound.”

**Trap:** “Low IPC always means memory?” → “No—locks and kernel overhead can also stall.”

---

## 18) “You suspect IO wait. What do you run and how do you interpret it?”

**Ideal answer:**
“I use `vmstat 1` and `iostat -x 1`. High `wa` and high `%util/await` indicate disk bottleneck. Then I use `pidstat -d 1` to find which process is doing IO. Fix is reducing sync IO, caching, batching, or faster storage.”

**Trap:** “CPU low but load high?” → “Could be IO wait or D-state tasks.”

---

## 19) “Disk is full but I can’t find big files with `du`. What now?”

**Ideal answer:**
“That often means deleted files are still open by a process. I run `lsof +L1` to find open-but-deleted files consuming space. Fix: restart the offending process or rotate logs properly.”

**Trap:** “Why does `df` still show full?” → “Blocks aren’t freed until the last FD closes.”

---

## 20) “Explain memory ballooning and how it looks inside a VM.”

**Ideal answer:**
“Ballooning is when the hypervisor reclaims memory from the guest using a balloon driver; the guest loses usable memory and may swap more. Inside the VM I’d see reduced MemAvailable, rising swap activity in `vmstat si/so`, and latency spikes even if app RSS didn’t grow.”

**Trap:** “What’s the ‘Google-y’ interpretation?” → “Consider noisy neighbor/host contention, not just app leaks.”

---

# Bonus: 10 lightning follow-up traps (very common)

1. **“Is a page fault always bad?”** → No; demand paging and copy-on-write are normal.
2. **“Segfault is a syscall?”** → No; it’s a signal delivered after invalid memory access.
3. **“Nice vs chrt?”** → nice affects CFS weight; chrt sets realtime policies.
4. **“High load average but CPU idle?”** → often IO wait or blocked tasks in D state.
5. **“Why might `curl` work but browser fail?”** → cookies, CORS, proxies, HSTS, SNI/Host differences.
6. **“What is `D` state?”** → uninterruptible sleep, usually waiting on IO.
7. **“What does `sy` mean in vmstat?”** → CPU time in kernel mode.
8. **“Why are too many threads bad?”** → context switch overhead + cache thrash.
9. **“Why does buffering exist?”** → performance; fewer syscalls.
10. **“What’s the first thing you do in an incident?”** → classify + get one strong proof signal.

---

