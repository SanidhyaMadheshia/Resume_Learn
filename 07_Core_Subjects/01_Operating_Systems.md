# Operating Systems — Complete Interview Guide

Written for backend SDE intern / new-grad rounds (Amazon, Google, Flipkart, Zomato/Eternal, etc.).
Read it top to bottom once. Before the interview, read only **Section 13 (Top 40)** and **Section 14 (Last-minute revision)**.

How each topic is laid out:
**Simple explanation → Key points → Diagram/table → Code → Interview tip / common trap.**

Symbols: ⭐ = asked very often. 🔗 = **Project link** (say this in the interview — it shows you used the idea for real).

All full C++ programs (not the 1–2 line snippets) compile with `g++ -std=c++20 -pthread file.cpp` (or `clang++`). POSIX examples (fork, pipe) need Linux or macOS.

---

## Table of Contents

1. [OS Basics](#1-os-basics)
2. [Processes](#2-processes)
3. [Threads](#3-threads)
4. [CPU Scheduling](#4-cpu-scheduling)
5. [Synchronization](#5-synchronization)
6. [Classic Synchronization Problems](#6-classic-synchronization-problems)
7. [Deadlocks](#7-deadlocks)
8. [Memory Management](#8-memory-management)
9. [Virtual Memory and Page Replacement](#9-virtual-memory-and-page-replacement)
10. [Storage and File Systems](#10-storage-and-file-systems)
11. [I/O Systems](#11-io-systems)
12. [Backend-Relevant Extras](#12-backend-relevant-extras)
13. [Most Asked OS Interview Questions (Top 40)](#13-most-asked-os-interview-questions-top-40)
14. [Last-minute Revision](#14-last-minute-revision)

---

## 1. OS Basics

### 1.1 What is an Operating System?

**Simple explanation**
An OS is the **manager of the computer**. Think of a hotel manager. Guests (programs) want rooms (memory), staff time (CPU), and services (disk, network). The manager decides who gets what, keeps guests out of each other's rooms, and gives one simple front desk to talk to.

**Key points**
- **Resource manager**: shares CPU, memory, disk, and devices between programs.
- **Abstraction provider**: you call `read(file)`; you do not need to know how the SSD works.
- **Protection**: one program cannot read another program's memory.
- Main parts: process management, memory management, file system, I/O management, security.

### 1.2 Types of Operating Systems

| Type | Simple idea | Example |
|---|---|---|
| **Batch** | Jobs are grouped and run one after another. No user interaction. | Old mainframes |
| **Multiprogramming** | Keep many programs in memory. When one waits for I/O, run another. Goal: CPU never idle. | Early UNIX idea |
| **Multitasking / Time-sharing** | CPU switches between programs very fast (time slices), so every user feels the machine is theirs. Goal: fast response. | Linux, Windows, macOS |
| **Multiprocessing** | More than one CPU/core runs programs truly in parallel. | Any modern server |
| **Real-time (RTOS)** | Must finish tasks before a **deadline**. *Hard* real-time: missing a deadline = failure (airbag). *Soft*: late is bad but OK (video streaming). | FreeRTOS, VxWorks |
| **Distributed** | Many machines look like one system. | Cluster OS ideas |
| **Embedded** | Small OS inside a device. | Router, washing machine |

**Interview tip:** People mix up *multiprogramming* and *multitasking*. Multiprogramming = "do not let CPU sit idle" (switch on I/O). Multitasking = multiprogramming **plus** time slices so that users get fast response.

### 1.3 Kernel — Monolithic vs Microkernel

**Simple explanation**
The **kernel** is the core of the OS. It is the part that always stays in memory and has full power over the hardware. Analogy: the kernel is the **bank vault staff** — only they can touch the money directly.

| | Monolithic kernel | Microkernel | Hybrid |
|---|---|---|---|
| Idea | Everything (scheduler, memory, file system, drivers) runs inside the kernel | Only minimum inside kernel (scheduling, basic memory, IPC). Drivers & file systems run as user-space servers | Mix of both |
| Speed | Fast (direct function calls) | Slower (messages between servers = more IPC) | Medium |
| Safety | One buggy driver can crash the whole system | A crashed driver can be restarted | Medium |
| Example | **Linux**, old UNIX | MINIX, QNX, seL4 | Windows NT, macOS (XNU) |

**Key points**
- Linux is monolithic but **modular**: you can load/unload drivers (kernel modules) at runtime.
- **Trap:** "Linux is a microkernel because it has modules" — wrong. Modules still run in kernel mode.

### 1.4 User Mode vs Kernel Mode

**Simple explanation**
The CPU has a **mode bit**. In **user mode**, a program is like a customer in a bank: it can only fill forms. In **kernel mode**, code is like bank staff: it can open the vault (touch hardware, change page tables, disable interrupts).

**Key points**
- Your normal code (C++, Go, Java) runs in **user mode**.
- Privileged instructions (I/O, changing memory maps, halting CPU) work only in **kernel mode**. If user code tries, the CPU raises an exception and the OS usually kills the program.
- Ways to enter kernel mode: **system call**, **interrupt** (device needs attention), **exception/trap** (divide by zero, page fault).

```
 User mode      [ your program ] --calls read()--> [ libc wrapper ]
 ------------------------------------------------------|------------  (trap / syscall instruction)
 Kernel mode                                 [ sys_read() in kernel ] --> disk driver
```

### 1.5 System Calls

**Simple explanation**
A **system call** is the **front desk window** of the kernel. User programs cannot walk into the vault; they hand a request through the window. Example: `read()`, `write()`, `open()`, `fork()`.

**Steps of a system call**
1. Program calls a library wrapper (e.g. `read()` in libc).
2. Wrapper puts the **syscall number** and arguments in registers.
3. Executes a special instruction (`syscall` on x86-64) → CPU switches to kernel mode.
4. Kernel looks up the handler in the **syscall table** and runs it.
5. Result goes back in a register; CPU returns to user mode.

**Categories (with Linux examples)**

| Category | Examples |
|---|---|
| Process control | `fork`, `exec`, `exit`, `wait`, `kill` |
| File management | `open`, `read`, `write`, `close`, `lseek` |
| Device management | `ioctl`, `read`, `write` |
| Information | `getpid`, `time`, `uname` |
| Communication | `pipe`, `socket`, `send`, `recv`, `shmget`, `mmap` |

**Interview tip:** A system call is **more expensive** than a normal function call (mode switch, saving registers, security checks), but much cheaper than a context switch. That is why buffered I/O exists: `printf` collects many characters and makes one `write()` call.

### 1.6 Booting (short)

Power on → firmware (**BIOS/UEFI**) runs self-test → loads **bootloader** (e.g. GRUB) → bootloader loads **kernel** into memory → kernel sets up memory, drivers → starts the first user process (**init / systemd**, PID 1) → systemd starts services and login.

---

## 2. Processes

### 2.1 Program vs Process

**Simple explanation**
A **program** is a recipe written on paper (a file on disk, passive). A **process** is the recipe **being cooked** right now (a running program, active), with its own kitchen, ingredients, and current step.

**Memory layout of a process**

```
 High address  +------------------+
               |      Stack       |  function calls, local variables (grows down)
               |        |         |
               |        v         |
               |                  |
               |        ^         |
               |        |         |
               |       Heap       |  new / malloc (grows up)
               +------------------+
               |   BSS (uninit)   |  global variables without a value
               |   Data (init)    |  global variables with a value
               +------------------+
 Low address   |   Text (code)    |  machine instructions (read-only)
               +------------------+
```

### 2.2 Process States

**Simple explanation**
A process is like a patient in a hospital: arrives (new), waits in the waiting room (ready), is with the doctor (running), is sent for a blood test and must wait for the result (waiting), and finally leaves (terminated).

```
            admitted          scheduler picks
   [New] ------------> [Ready] ---------------> [Running] ----exit----> [Terminated]
                         ^  ^                      |  |
                         |  +----- interrupt ------+  |
                         |      (time slice over)     |
                         |                            | I/O or event wait
                         +-------- [Waiting] <--------+
                          I/O done
```

**Key points**
- **Ready**: can run, waiting only for CPU.
- **Waiting / Blocked**: cannot run even if CPU is free (waiting for disk, network, lock).
- Some books add **Suspended Ready / Suspended Blocked**: process swapped out to disk (medium-term scheduler).
- **Trap:** A process goes Waiting → **Ready**, never Waiting → Running directly.

**Schedulers**
| Scheduler | Job | How often |
|---|---|---|
| Long-term (job scheduler) | Which new jobs enter memory (controls degree of multiprogramming) | Rare |
| Short-term (CPU scheduler) | Which ready process gets CPU next | Very often (ms) |
| Medium-term | Swap processes out/in to reduce memory load | Sometimes |

### 2.3 PCB (Process Control Block)

**Simple explanation**
The PCB is the process's **ID card + bookmark**. When the OS pauses a process, it writes down exactly where it stopped, so it can resume later as if nothing happened.

**PCB contains**
- PID, parent PID, state
- **Program counter** (next instruction) and CPU **registers**
- Scheduling info (priority, queue pointers)
- Memory info (page table pointer, limits)
- Open file table, I/O status
- Accounting (CPU time used)

In Linux the PCB is `struct task_struct`.

### 2.4 Context Switch ⭐

**Simple explanation**
A **context switch** is when the CPU stops running process A and starts process B. Like a teacher who checks one student's notebook, puts a bookmark in it, and picks up another student's notebook.

**Steps**
1. Interrupt or syscall enters kernel.
2. Save A's registers and program counter into A's PCB.
3. Scheduler picks B.
4. Load B's registers from B's PCB; switch page table (if B is a different process).
5. Return to user mode, B continues.

**Why it is expensive**
- **Direct cost**: saving/restoring registers, kernel code (a few microseconds).
- **Indirect cost (bigger)**: CPU caches and the **TLB** are now full of A's data. B suffers many cache/TLB misses at first ("cold cache").
- Thread switch inside the **same process** is cheaper: no page-table switch, so the TLB stays useful.

**Interview tip:** Context switch time is **pure overhead** — no useful work happens. That is why thread-per-request servers struggle at very high concurrency and why event loops (epoll) and goroutines exist.

### 2.5 fork(), exec(), wait() ⭐

**Simple explanation**
- `fork()` = **photocopy** the current process. Now there are two almost identical processes.
- `exec()` = **replace** the photocopy's content with a totally new program. Same PID, new code.
- `wait()` = the parent **waits** for the child to finish and collects its exit code.

This is exactly how a shell runs `ls`: shell forks, child execs `ls`, shell waits.

**Key points**
- `fork()` returns **twice**: `0` in the child, the **child's PID** in the parent, `-1` on error.
- After fork, parent and child have **separate** memory (changing a variable in the child does not change it in the parent). Linux uses **copy-on-write** so the copy is lazy and cheap (see 9.9).
- `exec()` does **not** return on success. Code after `exec` runs only if exec failed.
- `n` calls of `fork()` in a row → `2^n` processes (`2^n − 1` new children).

**Code: fork + exec + wait**

```cpp
// fork_exec_wait.cpp  — compile: g++ -std=c++20 fork_exec_wait.cpp
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <cstdio>

int main() {
    std::printf("Parent: my PID is %d\n", getpid());
    std::fflush(stdout);              // empty the print buffer BEFORE fork, else it gets copied and printed twice

    pid_t pid = fork();               // make a copy of this process
    if (pid < 0) {                    // fork failed (too many processes, no memory)
        std::perror("fork");
        return 1;
    }

    if (pid == 0) {                   // ---- this branch runs in the CHILD ----
        std::printf("Child: my PID is %d, my parent is %d\n", getpid(), getppid());
        std::fflush(stdout);
        char* args[] = {(char*)"ls", (char*)"-l", nullptr};
        execvp("ls", args);           // replace child with the "ls -l" program
        std::perror("execvp");        // we reach here ONLY if exec failed
        _exit(127);                   // exit child without running parent's cleanup
    }

    // ---- this part runs in the PARENT ----
    int status = 0;
    waitpid(pid, &status, 0);         // wait for that child; prevents a zombie
    if (WIFEXITED(status))
        std::printf("Parent: child %d exited with code %d\n", pid, WEXITSTATUS(status));
    return 0;
}
```

**Common trap questions**

```cpp
fork(); fork(); fork();
printf("hi\n");        // prints 8 times (2^3 processes)
```

```cpp
fork() && fork() || fork();   // total processes = 5
```
Why 5: in the first child, the first `fork()` returns 0 → `&&` skips the second fork → the third fork runs (2 processes). In the parent, the second fork runs. Its parent gets non-zero → `||` skips the third. Its child gets 0 → third fork runs (2 processes). Total = parent + child of 2nd fork + its child + first child + its child = **5**.

### 2.6 Zombie and Orphan Processes ⭐

**Simple explanation**
- **Zombie**: the child is **dead**, but the parent has not "collected the body" (has not called `wait()`). Its entry (PID + exit code) stays in the process table. Like a hotel guest who checked out, but the room is still marked "occupied" until reception signs it off.
- **Orphan**: the **parent died first** while the child is still running. The child is adopted by `init`/`systemd` (PID 1), which will `wait()` for it later.

**Key points**
- A zombie uses almost no memory, but it holds a **PID**. Thousands of zombies can exhaust the PID table → no new processes can be created.
- Fix zombies: parent must call `wait()`/`waitpid()`, or handle `SIGCHLD`, or ignore `SIGCHLD` (then the kernel cleans up automatically).
- You **cannot kill a zombie** with `kill -9` — it is already dead. Kill or fix the **parent**.
- In `ps`, zombie state shows as `Z` (`<defunct>`).

### 2.7 Inter-Process Communication (IPC) ⭐

**Simple explanation**
Processes have separate memory (separate houses). IPC is how they **talk**: passing notes (pipes, message queues), sharing a common room (shared memory), phone calls (sockets), or ringing the doorbell (signals).

| Method | Simple analogy | Key facts | Speed |
|---|---|---|---|
| **Pipe** (anonymous) | One-way water pipe | One direction, byte stream, only between related processes (parent–child). `ls \| grep x` | Medium |
| **Named pipe (FIFO)** | Pipe with a name on the wall | Has a file name, so unrelated processes can use it | Medium |
| **Message queue** | Post-box | Messages with boundaries & types; kernel stores them | Medium |
| **Shared memory** | Shared whiteboard | Same physical memory mapped into both processes. **Fastest** (no kernel copy after setup) but **you must synchronize** (semaphore/mutex) | Fastest |
| **Socket** | Phone call | Works across machines (TCP/UDP) or locally (Unix domain socket). Basis of all networking | Slower, most flexible |
| **Signal** | Doorbell | Tiny notification with a number (`SIGINT`, `SIGKILL`, `SIGTERM`, `SIGCHLD`). No data | Very small |
| **Memory-mapped file** (`mmap`) | Shared notebook on disk | File mapped into memory; can be shared | Fast |

**Signals you must know**
| Signal | Meaning | Can be caught? |
|---|---|---|
| `SIGINT` (2) | Ctrl+C | Yes |
| `SIGTERM` (15) | Polite "please stop" (default of `kill`) | Yes — used for graceful shutdown |
| `SIGKILL` (9) | Force kill | **No** |
| `SIGSTOP` | Pause | **No** |
| `SIGCHLD` | A child stopped/exited | Yes |
| `SIGSEGV` | Invalid memory access (segfault) | Yes (but usually just crash) |

**Code: parent → child through a pipe**

```cpp
// pipe_demo.cpp — parent writes a message, child reads it
#include <sys/wait.h>
#include <unistd.h>
#include <cstdio>
#include <cstring>

int main() {
    int fd[2];                        // fd[0] = read end, fd[1] = write end
    if (pipe(fd) == -1) { std::perror("pipe"); return 1; }

    pid_t pid = fork();
    if (pid == 0) {                   // child: reader
        close(fd[1]);                 // child does not write, close write end
        char buf[100] = {0};
        ssize_t n = read(fd[0], buf, sizeof(buf) - 1);   // blocks until data arrives
        std::printf("Child got %zd bytes: %s\n", n, buf);
        std::fflush(stdout);          // _exit() does not flush buffers, so flush first
        close(fd[0]);
        _exit(0);
    }

    close(fd[0]);                     // parent does not read, close read end
    const char* msg = "hello from parent";
    write(fd[1], msg, std::strlen(msg));
    close(fd[1]);                     // closing tells the reader "no more data" (EOF)
    wait(nullptr);                    // reap child
    return 0;
}
```

**Interview tip:** "Which IPC is fastest?" → **Shared memory**, because after setup, data does not pass through the kernel. "Which works across machines?" → **Sockets**. Always add: "shared memory needs synchronization."

🔗 **Project link:** "In LocalArena the backend talks to clients over sockets (WebSockets on TCP). I learned that the **Linux socket buffers** under my app are a hidden queue — more in Section 11.6."

---

## 3. Threads

### 3.1 What is a Thread? Thread vs Process ⭐

**Simple explanation**
A process is a **house**. Threads are **people living in the house**. They share the kitchen, fridge, and living room (code, heap, global data, open files), but each person has their own **diary** (stack) and their own **current thought** (registers, program counter).

| | Process | Thread |
|---|---|---|
| Memory | Own address space | Shares process memory (code, heap, globals, files) |
| Own stuff | Everything | Only stack, registers, program counter, thread ID |
| Creation cost | Heavy (new page tables etc.) | Light |
| Context switch | Slower (page table + TLB flush) | Faster (same address space) |
| Communication | IPC (pipes, sockets, shared memory) | Shared variables (needs locks) |
| Crash | Isolated — other processes survive | One bad thread (segfault) kills the **whole process** |
| Example | Chrome uses a process per tab (safety) | A web server uses a thread pool |

🔗 **Project link:** "In Go, **goroutines** are even lighter than threads — about **2 KB initial stack**, scheduled by the Go runtime onto a few OS threads (an **M:N** model). My TTL chat server runs **2 goroutines per WebSocket connection**: a reader that runs handlers, and a writer that is the **only** goroutine that writes to the socket."

### 3.2 User-Level vs Kernel-Level Threads

| | User-level threads | Kernel-level threads |
|---|---|---|
| Managed by | A library in user space; kernel does not know about them | The OS kernel |
| Switch speed | Very fast (no syscall) | Slower (kernel involved) |
| Blocking | One thread does a blocking syscall → **whole process blocks** (kernel sees only one thread) | Only that thread blocks |
| Multi-core | Cannot run in parallel on many cores (if 1 kernel thread underneath) | True parallelism |
| Example | Old "green threads" | `std::thread`, pthreads on Linux, Java threads |

### 3.3 Multithreading Models

```
 Many-to-One (M:1)        One-to-One (1:1)          Many-to-Many (M:N)
 U U U U                   U   U   U                  U U U U U U
  \ | | /                  |   |   |                   \ | X | X /
    K                      K   K   K                     K   K   K
 fast, no parallelism    Linux, Windows, Java,      Go runtime (goroutines),
 one block blocks all    C++ std::thread            Erlang; best of both but complex
```

- **Many-to-one**: many user threads on one kernel thread.
- **One-to-one**: each user thread = one kernel thread. Simple, real parallelism, but thread creation cost limits you to thousands, not millions.
- **Many-to-many**: many user threads multiplexed on a smaller pool of kernel threads. Go: **G** (goroutine) runs on **M** (OS thread) through **P** (processor slot, count = `GOMAXPROCS`).

### 3.4 Thread Pool

**Simple explanation**
Creating a thread for every request is like hiring a new waiter for every customer and firing him after the meal. A **thread pool** keeps a fixed team of waiters who take orders from a queue.

**Key points**
- Saves creation/destruction cost.
- **Limits** concurrency, so the server does not die from 10,000 threads.
- Used by Java `ExecutorService`, Tomcat, database connection pools (same idea for connections).

**Interview tip — Concurrency vs Parallelism:** *Concurrency* = dealing with many tasks at once (switching between them, even on 1 core). *Parallelism* = doing many tasks at the **same instant** (needs multiple cores). One cook handling 3 dishes = concurrency. Three cooks = parallelism.

---

## 4. CPU Scheduling

### 4.1 Basics and Terms ⭐

**Simple explanation**
Many processes are ready, but a core can run only one at a time. The **CPU scheduler** decides **who runs next**. Like a doctor's receptionist deciding the order of patients.

**Two kinds of bursts:** a process alternates between **CPU burst** (computing) and **I/O burst** (waiting for disk/network). *CPU-bound* = long CPU bursts (video encoding). *I/O-bound* = short CPU bursts, lots of waiting (web server).

**Preemptive vs Non-preemptive**
- **Non-preemptive**: once a process gets the CPU, it keeps it until it finishes or blocks. (FCFS, SJF, non-preemptive Priority)
- **Preemptive**: the OS can **take the CPU away** (timer interrupt, or a more important process arrived). (RR, SRTF, preemptive Priority, CFS)

**Formulas — memorize these**

| Term | Formula | Plain meaning |
|---|---|---|
| Arrival Time (AT) | given | When the process came |
| Burst Time (BT) | given | CPU time it needs |
| Completion Time (CT) | from Gantt chart | When it finished |
| **Turnaround Time (TAT)** | `CT − AT` | Total time in the system |
| **Waiting Time (WT)** | `TAT − BT` | Time spent waiting in the ready queue |
| **Response Time (RT)** | `first time it gets CPU − AT` | How long until it first ran |
| Throughput | processes finished / time | Work done per unit time |
| CPU utilization | busy time / total time | % of time CPU is busy |

Goals: maximize CPU utilization and throughput; minimize turnaround, waiting, and response time.

**Dispatcher**: the kernel code that actually gives the CPU to the chosen process (context switch, switch to user mode, jump to the right instruction). The time it takes = **dispatch latency**.

### 4.2 Scheduling Algorithms

| Algorithm | How it picks | Preemptive? | Good | Bad |
|---|---|---|---|---|
| **FCFS** (First Come First Serve) | Earliest arrival | No | Simple, fair in order | **Convoy effect**: one long job makes all short jobs wait (like a truck on a 1-lane road) |
| **SJF** (Shortest Job First) | Smallest burst | No | **Optimal average waiting time** (for non-preemptive) | Needs to know burst time in advance; long jobs may **starve** |
| **SRTF** (Shortest Remaining Time First) | Smallest *remaining* time; new shorter job preempts | Yes | Optimal average waiting time overall | Starvation, many context switches, burst must be predicted |
| **Priority** | Highest priority | Either | Important work first | **Starvation** of low priority → fix: **aging** |
| **Round Robin (RR)** | FIFO queue, each gets a **time quantum** then goes to back | Yes | Fair, good **response time**, no starvation | Quantum too big → becomes FCFS; too small → too many context switches |
| **Multilevel Queue (MLQ)** | Separate queues (system, interactive, batch), each with its own algorithm; fixed priority between queues | Yes | Different treatment for different jobs | Process stuck in its queue forever; lower queues may starve |
| **Multilevel Feedback Queue (MLFQ)** | Like MLQ but processes **move** between queues based on behavior | Yes | Learns: I/O-bound jobs stay high, CPU hogs sink. No need to know burst time | Complex to tune |
| **Linux CFS** | Lowest **virtual runtime** | Yes | Fair share for all | — (see 4.5) |

**Key points**
- **How do we know burst time for SJF?** We **predict** with exponential averaging: `τ(n+1) = α·t(n) + (1−α)·τ(n)` where `t(n)` = actual last burst, `τ(n)` = last prediction, `α` usually 0.5.
- **Aging** = slowly increase the priority of a process the longer it waits. Like a queue at a government office where elderly people waiting a long time are moved forward. It fixes starvation.
- RR rule of thumb: quantum should be larger than most CPU bursts (about 80% of bursts finish within one quantum), and much larger than context-switch time. Typical: 10–100 ms.
- **MLFQ rules (simple form):** new job enters the top queue (small quantum). If it uses its full quantum → moved **down** (it is CPU-hungry). If it gives up CPU early (I/O) → stays. Periodically **boost** all jobs to the top to avoid starvation.

### 4.3 Fully Worked Example ⭐

Same 4 processes for every algorithm:

| Process | Arrival (AT) | Burst (BT) |
|---|---|---|
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 1 |
| P4 | 3 | 2 |

#### (a) FCFS

Order of arrival: P1, P2, P3, P4.

Gantt chart:

| P1 | P2 | P3 | P4 |
|---|---|---|---|
| 0 → 5 | 5 → 8 | 8 → 9 | 9 → 11 |

| Process | AT | BT | CT | TAT = CT − AT | WT = TAT − BT |
|---|---|---|---|---|---|
| P1 | 0 | 5 | 5 | 5 | 0 |
| P2 | 1 | 3 | 8 | 7 | 4 |
| P3 | 2 | 1 | 9 | 7 | 6 |
| P4 | 3 | 2 | 11 | 8 | 6 |
| **Average** | | | | **27/4 = 6.75** | **16/4 = 4.00** |

See the **convoy effect**: P3 needs only 1 unit but waits 6.

#### (b) SJF (non-preemptive)

- t=0: only P1 has arrived → P1 runs 0–5 (cannot be stopped).
- t=5: P2 (3), P3 (1), P4 (2) are waiting → shortest first: P3, then P4, then P2.

| P1 | P3 | P4 | P2 |
|---|---|---|---|
| 0 → 5 | 5 → 6 | 6 → 8 | 8 → 11 |

| Process | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|
| P1 | 0 | 5 | 5 | 5 | 0 |
| P2 | 1 | 3 | 11 | 10 | 7 |
| P3 | 2 | 1 | 6 | 4 | 3 |
| P4 | 3 | 2 | 8 | 5 | 3 |
| **Average** | | | | **24/4 = 6.00** | **13/4 = 3.25** |

#### (c) SRTF (preemptive SJF)

Check at every arrival: if the new process has a smaller remaining time, it takes the CPU.
- t=0: P1 starts (remaining 5).
- t=1: P2 arrives (3) < P1 remaining (4) → **P2 preempts P1**.
- t=2: P3 arrives (1) < P2 remaining (2) → **P3 preempts P2**. P3 runs 2–3, done.
- t=3: P4 arrives (2). Remaining: P1=4, P2=2, P4=2. Tie P2/P4 → earlier arrival wins → P2 runs 3–5, done.
- t=5: P4 (2) runs 5–7. Then P1 (4) runs 7–11.

| P1 | P2 | P3 | P2 | P4 | P1 |
|---|---|---|---|---|---|
| 0 → 1 | 1 → 2 | 2 → 3 | 3 → 5 | 5 → 7 | 7 → 11 |

| Process | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|
| P1 | 0 | 5 | 11 | 11 | 6 |
| P2 | 1 | 3 | 5 | 4 | 1 |
| P3 | 2 | 1 | 3 | 1 | 0 |
| P4 | 3 | 2 | 7 | 4 | 2 |
| **Average** | | | | **20/4 = 5.00** | **9/4 = 2.25** |

#### (d) Round Robin, quantum = 2

Rule used (standard): when a process's quantum ends at time t, processes that **arrived by t are added to the queue first**, then the preempted process goes to the back.

| Time | Event | Ready queue after event |
|---|---|---|
| 0 | P1 runs 0–2 (rem 3). P2 (t=1), P3 (t=2) arrived | P2, P3, P1 |
| 2 | P2 runs 2–4 (rem 1). P4 arrived at 3 | P3, P1, P4, P2 |
| 4 | P3 runs 4–5 (done) | P1, P4, P2 |
| 5 | P1 runs 5–7 (rem 1) | P4, P2, P1 |
| 7 | P4 runs 7–9 (done) | P2, P1 |
| 9 | P2 runs 9–10 (done) | P1 |
| 10 | P1 runs 10–11 (done) | — |

| P1 | P2 | P3 | P1 | P4 | P2 | P1 |
|---|---|---|---|---|---|---|
| 0 → 2 | 2 → 4 | 4 → 5 | 5 → 7 | 7 → 9 | 9 → 10 | 10 → 11 |

| Process | AT | BT | CT | TAT | WT | Response (first run − AT) |
|---|---|---|---|---|---|---|
| P1 | 0 | 5 | 11 | 11 | 6 | 0 |
| P2 | 1 | 3 | 10 | 9 | 6 | 1 |
| P3 | 2 | 1 | 5 | 3 | 2 | 2 |
| P4 | 3 | 2 | 9 | 6 | 4 | 4 |
| **Average** | | | | **29/4 = 7.25** | **18/4 = 4.50** | **7/4 = 1.75** |

#### Comparison

| Algorithm | Avg TAT | Avg WT |
|---|---|---|
| FCFS | 6.75 | 4.00 |
| SJF | 6.00 | 3.25 |
| **SRTF** | **5.00** | **2.25** (best) |
| RR (q=2) | 7.25 | 4.50 |

**Lesson:** SRTF gives the best average waiting time. RR has worse averages, but **every process starts quickly** (good response time) and nobody starves — that is why interactive systems use RR-like ideas.

**Priority (quick example)** — add priorities P1=3, P2=1, P3=4, P4=2 (smaller number = higher priority), non-preemptive: P1 0–5 (only one at t=0), then P2 5–8, P4 8–10, P3 10–11.

**Common traps**
- Forgetting the CPU can be **idle** if no process has arrived yet (Gantt chart has a gap).
- In RR, different books order "new arrival vs preempted process" differently. **Say your rule out loud** to the interviewer.
- WT for preemptive algorithms is still `TAT − BT` (simplest way; do not try to sum gaps).

### 4.4 Code: Round Robin Simulator

```cpp
// round_robin.cpp — simulates RR and prints Gantt chart + CT/TAT/WT
#include <algorithm>
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

struct Proc {
    int id, at, bt;        // id, arrival time, burst time
    int rem = 0;           // remaining burst
    int ct = 0;            // completion time
    int firstRun = -1;     // when it first got the CPU
};

int main() {
    vector<Proc> p = {{1, 0, 5}, {2, 1, 3}, {3, 2, 1}, {4, 3, 2}};
    int quantum = 2;
    int n = p.size();
    for (auto& x : p) x.rem = x.bt;
    sort(p.begin(), p.end(), [](const Proc& a, const Proc& b) { return a.at < b.at; });

    queue<int> rq;         // ready queue of indexes into p
    int time = 0, next = 0, done = 0;

    // helper: move everyone who has arrived by 'time' into the ready queue
    auto admit = [&]() {
        while (next < n && p[next].at <= time) rq.push(next++);
    };

    admit();
    cout << "Gantt: ";
    while (done < n) {
        if (rq.empty()) {                 // CPU idle: jump to next arrival
            time = p[next].at;
            admit();
            continue;
        }
        int i = rq.front(); rq.pop();
        if (p[i].firstRun == -1) p[i].firstRun = time;

        int run = min(quantum, p[i].rem); // run one quantum or until done
        cout << "| P" << p[i].id << " " << time << "-" << time + run << " ";
        time += run;
        p[i].rem -= run;

        admit();                          // new arrivals go in BEFORE the preempted one
        if (p[i].rem > 0) rq.push(i);     // not finished: back of the queue
        else { p[i].ct = time; done++; }  // finished
    }
    cout << "|\n\nP  AT BT CT TAT WT RT\n";

    double sumTat = 0, sumWt = 0;
    for (auto& x : p) {
        int tat = x.ct - x.at, wt = tat - x.bt, rt = x.firstRun - x.at;
        sumTat += tat; sumWt += wt;
        cout << "P" << x.id << " " << x.at << "  " << x.bt << "  " << x.ct
             << "  " << tat << "   " << wt << "  " << rt << "\n";
    }
    cout << "Avg TAT = " << sumTat / n << ", Avg WT = " << sumWt / n << "\n";
}
```

Output matches the worked example: `Avg TAT = 7.25, Avg WT = 4.5`.

### 4.5 Linux CFS (Completely Fair Scheduler) and Go's Scheduler

**Simple explanation**
CFS tries to give every process a **fair share** of CPU. Imagine every process has a **stopwatch** (virtual runtime, `vruntime`) that ticks while it runs. The scheduler always picks the process whose stopwatch shows the **least time** — the one that has been treated most unfairly so far.

**Key points**
- Ready tasks are stored in a **red-black tree** sorted by `vruntime`. The **leftmost** node (smallest vruntime) runs next → picking is O(1) with a cached pointer, inserting is O(log n).
- **Nice value** (−20 to +19) changes the **weight**. High-priority (low nice) tasks' stopwatches tick **slower**, so they get more CPU.
- No fixed time quantum: the time slice is computed from the number of tasks (target latency / number of tasks, with a minimum).
- I/O-bound tasks sleep a lot → their vruntime stays low → when they wake they run quickly → **good interactivity** for free.
- Since Linux **6.6**, CFS was replaced by **EEVDF** (Earliest Eligible Virtual Deadline First) — same fairness idea plus better latency. Saying this shows you are up to date.
- Real-time classes (`SCHED_FIFO`, `SCHED_RR`) run before normal CFS tasks.

**Go scheduler (GMP)** — *G* = goroutine, *M* = OS thread, *P* = logical processor (count = `GOMAXPROCS`, default = number of cores). Each P has a local run queue; idle Ps **steal work** from others. Since Go 1.14, goroutines are **asynchronously preempted** (a tight loop cannot hog a thread forever).

**Interview tip:** If asked "Which scheduling algorithm does Linux use?", answer: "CFS — fair scheduling using a red-black tree on virtual runtime; newer kernels (6.6+) use EEVDF. Real OSes approximate MLFQ ideas: interactive tasks get priority automatically."

🔗 **Project link:** "LocalArena runs a fixed **20 Hz tick** (every 50 ms). At 1,000 players the tick p99 was **0.76 ms on 4 cores** — about 1.5% of the budget. Even at saturation (2,000 bots / 200 Hz / 1 CPU) the bottleneck was **fan-out** (sending snapshots), not the simulation itself."

---

## 5. Synchronization

### 5.1 Race Condition ⭐

**Simple explanation**
A **race condition** happens when two threads use the same data at the same time, at least one of them **writes**, and the final result depends on **who runs first**. Analogy: two people share one bank account with ₹100. Both check the balance at the same moment, both see ₹100, both withdraw ₹100. The bank lost ₹100.

**Why `counter++` is not safe** — it is really 3 steps:
```
1. LOAD  counter -> register     (read 5)
2. ADD   register + 1            (6)
3. STORE register -> counter     (write 6)
```
If thread A and thread B both do step 1 before either does step 3, both write 6. One increment is **lost**.

| Time | Thread A | Thread B | counter |
|---|---|---|---|
| 1 | load 5 | | 5 |
| 2 | | load 5 | 5 |
| 3 | add → 6 | | 5 |
| 4 | | add → 6 | 5 |
| 5 | store 6 | | 6 |
| 6 | | store 6 | **6 (should be 7)** |

**Code: race → fix with mutex → fix with atomic**

```cpp
// race_demo.cpp — compile WITHOUT optimization to see the race clearly:
//   g++ -std=c++20 -O0 -pthread race_demo.cpp
#include <atomic>
#include <iostream>
#include <mutex>
#include <thread>

const int N = 1'000'000;   // each thread adds 1 this many times; correct total = 2N

void withRace() {
    int counter = 0;
    auto work = [&] { for (int i = 0; i < N; i++) counter++; };  // NOT safe (data race = undefined behavior)
    std::thread t1(work), t2(work);
    t1.join(); t2.join();
    std::cout << "No lock : " << counter << " (expected " << 2 * N << ")\n";
}

void withMutex() {
    int counter = 0;
    std::mutex m;                          // the "bathroom key"
    auto work = [&] {
        for (int i = 0; i < N; i++) {
            std::lock_guard<std::mutex> lock(m);   // take key; auto-returned at end of scope
            counter++;                             // critical section
        }
    };
    std::thread t1(work), t2(work);
    t1.join(); t2.join();
    std::cout << "Mutex   : " << counter << "\n";
}

void withAtomic() {
    std::atomic<int> counter{0};           // hardware does load+add+store as ONE step
    auto work = [&] { for (int i = 0; i < N; i++) counter.fetch_add(1); };
    std::thread t1(work), t2(work);
    t1.join(); t2.join();
    std::cout << "Atomic  : " << counter.load() << "\n";
}

int main() {
    withRace();     // usually prints a number LESS than 2000000
    withMutex();    // always 2000000
    withAtomic();   // always 2000000, and faster than mutex for a single counter
}
```

**Key points**
- **Mutex** fixes it by allowing only one thread inside at a time.
- **Atomic** fixes it with a single CPU instruction (e.g. `lock xadd` on x86). Faster, but only for **simple operations on one variable**. For "update two things together", use a mutex.
- Tools to find races: **ThreadSanitizer** (`-fsanitize=thread` in C++), Go's **`-race`** flag.

🔗 **Project link:** "I avoid races in LocalArena with the **actor model** — each match's world state is owned by **one goroutine**, and all input reaches it through **channels**, so no locks are needed on game state. CI runs all **27 Go test suites with `-race`**."

### 5.2 Critical Section and its 3 Requirements ⭐

**Simple explanation**
The **critical section** is the part of code that touches shared data. Only one thread should be in it at a time — like a single-person **bathroom**.

```
do {
    entry section      // ask for permission (lock)
    CRITICAL SECTION   // use shared data
    exit section       // give up permission (unlock)
    remainder section  // other work
} while (true);
```

**3 requirements of a correct solution**
1. **Mutual Exclusion** — at most one process is inside the critical section.
2. **Progress** — if nobody is inside, and some want to enter, the decision cannot be postponed forever; only those who want to enter take part in deciding. (No one blocks the door while not even using it.)
3. **Bounded Waiting** — there is a limit on how many times others can enter before a waiting process gets its turn. (No starvation.)

Plus a general expectation: no assumption about the speed or number of CPUs.

**Trap:** "Progress" and "bounded waiting" are different. Progress = *someone* will get in. Bounded waiting = *I* will get in after a limited number of turns.

### 5.3 Peterson's Solution (2 processes)

**Simple explanation**
Two people at a door. Each raises a hand ("I want to go": `flag[i] = true`) and then politely says "**you go first**" (`turn = other`). If both say it at the same time, the one who said it **last** loses and waits. Fair and simple.

```cpp
// peterson.cpp — Peterson's algorithm for 2 threads (atomics give the memory ordering it needs)
#include <atomic>
#include <iostream>
#include <thread>

std::atomic<bool> flag[2] = {false, false};  // flag[i] = thread i wants to enter
std::atomic<int> turn{0};                     // whose turn it is to WAIT for the other
int counter = 0;                              // shared data

void lock(int i) {
    int j = 1 - i;               // the other thread
    flag[i] = true;              // I want to enter
    turn = j;                    // but you go first
    while (flag[j] && turn == j) { /* busy wait */ }
}

void unlock(int i) { flag[i] = false; }      // I am done

void work(int id) {
    for (int k = 0; k < 100000; k++) {
        lock(id);
        counter++;               // critical section
        unlock(id);
    }
}

int main() {
    std::thread a(work, 0), b(work, 1);
    a.join(); b.join();
    std::cout << counter << "\n";   // 200000
}
```

**Key points**
- Satisfies all 3 requirements (mutual exclusion, progress, bounded waiting).
- Only for **2** processes and uses **busy waiting**.
- **Trap:** On modern CPUs, Peterson's with **plain variables** can **fail**, because CPUs and compilers **reorder** memory operations. It works above only because `std::atomic` defaults to sequentially consistent ordering (memory barriers). This is a great point to mention.

### 5.4 Hardware Support: Test-and-Set, Compare-and-Swap

- **Test-and-Set (TAS)**: atomically "set to true and return the old value". If old value was false, you got the lock.
- **Compare-and-Swap (CAS)**: atomically "if value == expected, set it to new; tell me if it worked". Basis of **lock-free** data structures and `std::atomic::compare_exchange_strong`, Java `AtomicInteger`, and **optimistic locking** in databases (`UPDATE ... WHERE version = 5`).

### 5.5 Mutex ⭐

**Simple explanation**
A **mutex** (mutual exclusion lock) is the **bathroom key**. There is only one key. Take it to enter; others wait outside (sleeping). Only the person **who took the key** returns it.

**Key points**
- Has **ownership**: only the thread that locked it may unlock it.
- Waiting threads **sleep** (blocked) — no CPU wasted.
- In C++ always use RAII: `std::lock_guard`, `std::unique_lock`, `std::scoped_lock`. They unlock automatically even if an exception is thrown.
- **Recursive mutex**: the same thread can lock it again (count increments). Usually a sign of bad design.

### 5.6 Spinlock

**Simple explanation**
A **spinlock** is a mutex where the waiting thread does **not** sleep. It keeps asking "free yet? free yet?" in a loop. Like standing at the bathroom door and knocking non-stop.

```cpp
// spinlock.cpp — a minimal spinlock using atomic_flag (test-and-set)
#include <atomic>
#include <iostream>
#include <thread>

class SpinLock {
    std::atomic_flag f = ATOMIC_FLAG_INIT;
public:
    void lock()   { while (f.test_and_set(std::memory_order_acquire)) { /* spin */ } }
    void unlock() { f.clear(std::memory_order_release); }
};

SpinLock sl;
long total = 0;

int main() {
    auto work = [] { for (int i = 0; i < 100000; i++) { sl.lock(); total++; sl.unlock(); } };
    std::thread a(work), b(work);
    a.join(); b.join();
    std::cout << total << "\n";  // 200000
}
```

| | Mutex (sleeping lock) | Spinlock |
|---|---|---|
| Waiting thread | Sleeps (context switch) | Busy-loops (burns CPU) |
| Best when | Lock is held for a long time | Lock is held for a **very short** time **on multi-core** |
| Single core | Fine | Bad: spinner wastes the whole time slice while the owner cannot run |
| Used in | Application code | OS kernels, interrupt handlers (cannot sleep there) |

Real mutexes (Linux **futex**) are hybrid: spin a little, then sleep.

### 5.7 Semaphore ⭐

**Simple explanation**
A **semaphore** is an integer counter with two operations. Analogy: a **parking lot with N spaces** and a display board showing free spaces.
- `wait()` / `P()` / `down()` / `acquire()`: if free spaces > 0, take one (counter−−). Else wait.
- `signal()` / `V()` / `up()` / `release()`: leave, free a space (counter++), wake a waiting car.

```
wait(S):   while (S <= 0) sleep;  S--;      // done atomically
signal(S): S++;  wake one sleeper;
```

**Types**
- **Counting semaphore**: value 0..N. Limits how many can use a resource at once (N DB connections, N worker slots).
- **Binary semaphore**: value 0 or 1. Looks like a mutex, but has **no owner** — any thread can signal it. Good for **signaling** ("data is ready").

**Mutex vs Semaphore** ⭐

| | Mutex | Semaphore |
|---|---|---|
| What it is | A lock | A counter |
| Holders at once | 1 | N (counting) |
| Ownership | Yes — only locker unlocks | No — any thread can signal |
| Main use | Mutual exclusion | Limiting access to N resources, or **signaling** between threads |
| Analogy | Bathroom key | Parking lot with N spots |

**Code: counting semaphore limiting concurrent workers (C++20)**

```cpp
// semaphore_demo.cpp — 8 workers, but at most 3 run at the same time
#include <atomic>
#include <chrono>
#include <iostream>
#include <mutex>
#include <semaphore>
#include <thread>
#include <vector>

std::counting_semaphore<3> slots(3);   // 3 free "parking spots"
std::atomic<int> active{0};            // how many are inside right now
std::mutex printMu;                    // keep printed lines from mixing

void worker(int id) {
    slots.acquire();                   // wait for a free spot (counter--)
    int now = ++active;
    {
        std::lock_guard<std::mutex> g(printMu);
        std::cout << "Worker " << id << " started, active = " << now << "\n";
    }
    std::this_thread::sleep_for(std::chrono::milliseconds(100));  // pretend to call a DB
    --active;
    slots.release();                   // free the spot (counter++), wakes a waiter
}

int main() {
    std::vector<std::thread> ts;
    for (int i = 1; i <= 8; i++) ts.emplace_back(worker, i);
    for (auto& t : ts) t.join();
    // "active" never goes above 3
}
```

**Interview tip:** "Is a binary semaphore the same as a mutex?" → No. Same values, but a mutex has **ownership** (and often priority inheritance); a binary semaphore can be released by a different thread, so it is used for signaling.

### 5.8 Monitors and Condition Variables

**Simple explanation**
A **monitor** is a class where **only one thread can be inside any method at a time** (the lock is built in). Inside, threads can **wait** for a condition using a **condition variable**.
Analogy: a **doctor's room** with one door (lock). If the patient needs a blood report that isn't ready, they go to the **waiting bench** (condition variable) and **give up the room** so others can enter. When the report comes, the nurse **calls them** (notify).

**Key points**
- `wait(lock)`: atomically **releases the lock and sleeps**. When woken, it **re-acquires the lock** before returning.
- `notify_one()` / `notify_all()` (C++), `signal()` / `broadcast()` (pthreads), `notify()` / `notifyAll()` (Java).
- **Always wait in a loop** (or with a predicate): `cv.wait(lock, []{ return !q.empty(); });`. Reason: **spurious wakeups** (thread can wake up without notify) and another thread may have taken the item first.
- Java: every object is a monitor — `synchronized` + `wait()` / `notify()`.

**Trap:** Using `if` instead of `while` around `wait()`. Always say "while, because of spurious wakeups."

---

## 6. Classic Synchronization Problems

### 6.1 Producer–Consumer (Bounded Buffer) ⭐

**Simple explanation**
A **kitchen counter** that holds at most N plates. The **cook** (producer) puts plates; if the counter is full, the cook waits. The **waiter** (consumer) takes plates; if the counter is empty, the waiter waits.

**Rules**
- Producer must wait when buffer is **full**.
- Consumer must wait when buffer is **empty**.
- Only one should change the buffer at a time.

**Classic semaphore solution (pseudocode)**
```
semaphore mutex = 1;   // protects the buffer
semaphore empty = N;   // free slots
semaphore full  = 0;   // filled slots

Producer:               Consumer:
  wait(empty);            wait(full);
  wait(mutex);            wait(mutex);
  add item;               remove item;
  signal(mutex);          signal(mutex);
  signal(full);           signal(empty);
```
**Trap:** If the producer does `wait(mutex)` **before** `wait(empty)` and the buffer is full, it sleeps holding the mutex → consumer can never enter → **deadlock**. Order matters.

**Code: bounded buffer with mutex + condition_variable**

```cpp
// producer_consumer.cpp
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <queue>
#include <thread>

class BoundedBuffer {
    std::queue<int> q;
    size_t cap;
    std::mutex m;
    std::condition_variable notFull, notEmpty;
public:
    explicit BoundedBuffer(size_t c) : cap(c) {}

    void put(int x) {
        std::unique_lock<std::mutex> lock(m);
        notFull.wait(lock, [&] { return q.size() < cap; });  // sleep while full
        q.push(x);
        notEmpty.notify_one();                               // wake a consumer
    }

    int take() {
        std::unique_lock<std::mutex> lock(m);
        notEmpty.wait(lock, [&] { return !q.empty(); });     // sleep while empty
        int x = q.front(); q.pop();
        notFull.notify_one();                                // wake a producer
        return x;
    }
};

int main() {
    BoundedBuffer buf(3);                    // counter holds 3 plates max

    std::thread producer([&] {
        for (int i = 1; i <= 10; i++) buf.put(i);
        buf.put(-1);                         // special "poison pill" = no more items
    });

    std::thread consumer([&] {
        while (true) {
            int x = buf.take();
            if (x == -1) break;
            std::cout << "consumed " << x << "\n";
        }
    });

    producer.join();
    consumer.join();
}
```

**Same idea in Go (channels are built-in bounded buffers)**

```go
// producer_consumer.go — run: go run producer_consumer.go
package main

import (
	"fmt"
	"sync"
)

func main() {
	ch := make(chan int, 3) // buffered channel = bounded buffer of size 3
	var wg sync.WaitGroup

	// Producer: send blocks automatically when the buffer is full.
	go func() {
		for i := 1; i <= 10; i++ {
			ch <- i
		}
		close(ch) // tell consumers "no more items" (like the poison pill)
	}()

	// Two consumers: receive blocks automatically when the buffer is empty.
	for w := 1; w <= 2; w++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			for x := range ch { // loop ends when channel is closed and empty
				fmt.Printf("consumer %d got %d\n", id, x)
			}
		}(w)
	}
	wg.Wait()
}
```

> "Don't communicate by sharing memory; share memory by communicating." — Go proverb.

🔗 **Project link:** "A bounded buffer is exactly a **Go buffered channel** (or a Kafka topic between services). LocalArena's actor-per-match design is producer–consumer: client inputs are produced into the match's channel, and the single match goroutine consumes them each tick."

### 6.2 Readers–Writers ⭐

**Simple explanation**
A **notice board**. Many people can **read** it at the same time. But when someone is **writing** on it, nobody else may read or write.

**Variants**
- **First readers–writers (reader preference)**: readers never wait if others are reading → **writers can starve**.
- **Second (writer preference)**: once a writer is waiting, new readers wait → readers can starve.
- Fair versions use a queue/turnstile.

**Code: std::shared_mutex**

```cpp
// readers_writers.cpp
#include <iostream>
#include <map>
#include <mutex>
#include <shared_mutex>
#include <string>
#include <thread>
#include <vector>

class Config {
    std::map<std::string, std::string> data;
    mutable std::shared_mutex rw;              // read-write lock
public:
    std::string get(const std::string& k) const {
        std::shared_lock lock(rw);             // SHARED: many readers together
        auto it = data.find(k);
        return it == data.end() ? "" : it->second;
    }
    void set(const std::string& k, const std::string& v) {
        std::unique_lock lock(rw);             // EXCLUSIVE: one writer, no readers
        data[k] = v;
    }
};

int main() {
    Config cfg;
    cfg.set("mode", "v1");
    std::vector<std::thread> ts;
    for (int i = 0; i < 4; i++)                // 4 readers
        ts.emplace_back([&] { for (int k = 0; k < 1000; k++) cfg.get("mode"); });
    ts.emplace_back([&] { cfg.set("mode", "v2"); });   // 1 writer
    for (auto& t : ts) t.join();
    std::cout << cfg.get("mode") << "\n";      // v2
}
```

**Interview tip:** Use a read-write lock when reads are **much more** frequent than writes (config, cache). Go has `sync.RWMutex`, Java has `ReentrantReadWriteLock`. If writes are frequent, a plain mutex is often faster (RW locks have more overhead).

### 6.3 Dining Philosophers ⭐

**Simple explanation**
5 philosophers sit at a round table. Between each pair is **one fork** (5 forks total). To eat, a philosopher needs **both** the left and the right fork. If **everyone picks up their left fork at the same time**, everyone waits forever for the right fork → **deadlock**.

```
          P0
     F4        F0
  P4              P1
     F3        F1
     P3   F2   P2
```

**Solutions**
1. **Resource ordering** (best answer): number the forks; always pick up the **lower-numbered fork first**. This breaks **circular wait**.
2. Allow at most **4** philosophers to sit at once (semaphore of 4).
3. Odd philosophers pick left first, even pick right first.
4. Pick up both forks only if both are free (atomically, e.g. `std::scoped_lock`).

```cpp
// dining_philosophers.cpp — deadlock-free by always locking the lower-numbered fork first
#include <algorithm>
#include <chrono>
#include <iostream>
#include <mutex>
#include <thread>
#include <vector>

const int N = 5;
std::mutex forks[N];
std::mutex printMu;

void philosopher(int id) {
    int left = id, right = (id + 1) % N;
    int first = std::min(left, right);     // global order: low number first
    int second = std::max(left, right);

    for (int meal = 1; meal <= 3; meal++) {
        // think
        std::this_thread::sleep_for(std::chrono::milliseconds(10));

        std::lock_guard<std::mutex> a(forks[first]);
        std::lock_guard<std::mutex> b(forks[second]);
        {
            std::lock_guard<std::mutex> p(printMu);
            std::cout << "Philosopher " << id << " eats meal " << meal << "\n";
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(10));   // eat
    }   // forks released automatically here (end of loop body)
}

int main() {
    std::vector<std::thread> ts;
    for (int i = 0; i < N; i++) ts.emplace_back(philosopher, i);
    for (auto& t : ts) t.join();
    std::cout << "All done, no deadlock\n";
}
```

Why it works: philosopher 4 needs forks 4 and 0. With ordering, he picks **0 first**, same as philosopher 0. So the cycle "everyone holds left" can never form.

**Trap:** Solutions that fix deadlock may still allow **starvation** (one philosopher keeps losing). Mention it.

---

## 7. Deadlocks

### 7.1 What is a Deadlock? ⭐

**Simple explanation**
A **deadlock** is when a group of processes are all **waiting for each other**, so nobody can ever move. Analogy: two cars meet on a **one-lane bridge** from opposite sides. Neither will reverse. Both wait forever.

Another: Thread A holds lock 1 and wants lock 2. Thread B holds lock 2 and wants lock 1.

### 7.2 Four Necessary Conditions (Coffman) ⭐

All **four** must be true at the same time for a deadlock. Break **any one** → no deadlock.

| # | Condition | Plain meaning | Bridge analogy |
|---|---|---|---|
| 1 | **Mutual exclusion** | A resource can be used by only one at a time | Only one car fits on the bridge lane |
| 2 | **Hold and wait** | Holding one resource while waiting for another | Car stays on the bridge while waiting |
| 3 | **No preemption** | You cannot forcibly take a resource away | Nobody can tow a car away |
| 4 | **Circular wait** | A waits for B, B waits for C, …, last waits for A | Cars wait in a circle |

Memory trick: **"M-H-N-C" → "My Head Needs Coffee."**

### 7.3 Resource Allocation Graph (RAG)

**Simple explanation**
A picture of who holds what and who wants what.
- Circle = process, Square = resource (dots inside = instances).
- **Request edge**: Process → Resource ("I want it").
- **Assignment edge**: Resource → Process ("it is given to you").

```
   P1 ----request----> [R2]
   ^                    |
   | assigned           | assigned
   |                    v
  [R1] <----request---- P2

   Cycle: P1 -> R2 -> P2 -> R1 -> P1   => DEADLOCK (single instance resources)
```

**Key rules**
- **No cycle → no deadlock** (always).
- **Cycle + every resource has a single instance → deadlock** (definitely).
- **Cycle + multiple instances → maybe deadlock** (need to check further).
- For single-instance resources, simplify the RAG into a **wait-for graph** (only processes: P1 → P2 means P1 waits for P2). Cycle = deadlock.

### 7.4 Four Ways to Handle Deadlocks ⭐

| Strategy | Idea | Example |
|---|---|---|
| **Prevention** | Design so that one of the 4 conditions can **never** happen | Global lock order |
| **Avoidance** | Before granting a request, check if the system stays **safe** | Banker's algorithm |
| **Detection + Recovery** | Let deadlocks happen, find them, then fix | Databases (PostgreSQL, MySQL) |
| **Ignore (Ostrich algorithm)** | Pretend it never happens; reboot if it does | Linux, Windows for app-level locks |

### 7.5 Prevention — break one condition

| Condition to break | How | Problem |
|---|---|---|
| Mutual exclusion | Make resources shareable (read-only files) | Many resources (printer, lock) can't be shared |
| Hold and wait | Request **all** resources at once at start, or release everything before asking for more | Low utilization, starvation |
| No preemption | If you can't get the next resource, **release** what you hold (try-lock, then back off) | Works only for resources whose state can be saved (CPU, memory) |
| **Circular wait** | Give every resource a **number**; always acquire in **increasing order** | Need discipline. **Most practical** — used in real code |

### 7.6 Code: Deadlock Demo and Fix

```cpp
// deadlock_demo.cpp
//   ./a.out        -> runs the FIXED version (finishes)
//   ./a.out bad    -> runs the BAD version (usually hangs forever: Ctrl+C to stop)
#include <chrono>
#include <cstring>
#include <iostream>
#include <mutex>
#include <thread>

std::mutex accountA, accountB;

// BAD: the two threads lock in OPPOSITE order -> circular wait
void transferAtoB_bad() {
    std::lock_guard<std::mutex> l1(accountA);
    std::this_thread::sleep_for(std::chrono::milliseconds(50)); // make the bad timing very likely
    std::lock_guard<std::mutex> l2(accountB);                  // waits for B (held by other thread)
    std::cout << "A->B done\n";
}
void transferBtoA_bad() {
    std::lock_guard<std::mutex> l1(accountB);
    std::this_thread::sleep_for(std::chrono::milliseconds(50));
    std::lock_guard<std::mutex> l2(accountA);                  // waits for A -> DEADLOCK
    std::cout << "B->A done\n";
}

// FIX 1: std::scoped_lock locks BOTH using a deadlock-avoidance algorithm (all or nothing)
void transferAtoB_good() {
    std::scoped_lock lock(accountA, accountB);
    std::cout << "A->B done\n";
}
// FIX 2: fixed global order — every thread always locks A first, then B
void transferBtoA_good() {
    std::lock_guard<std::mutex> l1(accountA);   // A first, even though we move money B->A
    std::lock_guard<std::mutex> l2(accountB);
    std::cout << "B->A done\n";
}

int main(int argc, char** argv) {
    bool bad = argc > 1 && std::strcmp(argv[1], "bad") == 0;
    std::thread t1(bad ? transferAtoB_bad : transferAtoB_good);
    std::thread t2(bad ? transferBtoA_bad : transferBtoA_good);
    t1.join(); t2.join();
    std::cout << "finished without deadlock\n";
}
```

**Interview tip:** In real code, a common trick for "lock two objects" is to **lock by ID order** (smaller account ID first). Say that when asked about bank transfers.

🔗 **Project link (strong — tell this story):** "I hit a **real deadlock in PostgreSQL** in LocalArena — **200 concurrent buyers, 20 items, 0 succeeded**. Fix: changed the lock mode, used a **conditional update**, and **always lock rows in a fixed global order** — that is *prevention* by breaking **circular wait**. With the conditional update, the 200-buyer / 20-item run took **129 ms** (vs 832–984 ms for the other approaches). Any remaining deadlocks that the database detects are **retried with jittered backoff** — that is *detection + recovery*."

### 7.7 Avoidance — Safe State and Banker's Algorithm ⭐

**Simple explanation**
A **banker** has limited cash. Each customer has declared a **maximum loan** they may need. The banker gives cash only if, after giving, he can still find **some order** in which every customer can get their full maximum and repay. If such an order exists, the state is **safe**.

- **Safe state**: there exists a **safe sequence** — an order where every process can get what it still needs and finish.
- **Unsafe state**: no such order. Unsafe does **not** mean deadlock yet, but deadlock **may** happen.
- Safe ⊂ no deadlock. Deadlock ⊂ unsafe.

**Data structures** (n processes, m resource types)
- `Available[m]` — free instances now.
- `Max[n][m]` — max each process may ever need.
- `Allocation[n][m]` — currently held.
- `Need[n][m] = Max − Allocation` — still needed.

**Safety algorithm**
1. `Work = Available`, `Finish[i] = false` for all.
2. Find a process `i` with `Finish[i] == false` and `Need[i] ≤ Work` (every resource).
3. If found: pretend it finishes and returns everything: `Work += Allocation[i]`, `Finish[i] = true`, go to step 2.
4. If all `Finish` are true → **safe**. Else → **unsafe**.

Time: O(n² · m).

**Resource-request algorithm** (process `i` asks for `Request[i]`)
1. If `Request > Need[i]` → error (asked more than declared).
2. If `Request > Available` → must wait.
3. **Pretend** to grant: `Available −= Request`, `Allocation[i] += Request`, `Need[i] −= Request`.
4. Run safety check. Safe → grant for real. Unsafe → undo and make it wait.

#### Worked example

5 processes, 3 resource types A, B, C. Total instances: A = 10, B = 5, C = 7.

| Process | Allocation (A B C) | Max (A B C) | Need = Max − Alloc (A B C) |
|---|---|---|---|
| P0 | 0 1 0 | 7 5 3 | 7 4 3 |
| P1 | 2 0 0 | 3 2 2 | 1 2 2 |
| P2 | 3 0 2 | 9 0 2 | 6 0 0 |
| P3 | 2 1 1 | 2 2 2 | 0 1 1 |
| P4 | 0 0 2 | 4 3 3 | 4 3 1 |

Total allocated = (7, 2, 5). **Available = (10,5,7) − (7,2,5) = (3, 3, 2).**

**Safety check (scan P0 → P4 repeatedly)**

| Step | Check | Need ≤ Work? | Work after (Work + Allocation) |
|---|---|---|---|
| start | | | (3, 3, 2) |
| 1 | P0 needs (7,4,3) | No | (3, 3, 2) |
| 2 | P1 needs (1,2,2) | **Yes** → P1 finishes | (3,3,2)+(2,0,0) = **(5, 3, 2)** |
| 3 | P2 needs (6,0,0) | No (6 > 5) | (5, 3, 2) |
| 4 | P3 needs (0,1,1) | **Yes** → P3 finishes | (5,3,2)+(2,1,1) = **(7, 4, 3)** |
| 5 | P4 needs (4,3,1) | **Yes** → P4 finishes | (7,4,3)+(0,0,2) = **(7, 4, 5)** |
| 6 | P0 needs (7,4,3) | **Yes** → P0 finishes | (7,4,5)+(0,1,0) = **(7, 5, 5)** |
| 7 | P2 needs (6,0,0) | **Yes** → P2 finishes | (7,5,5)+(3,0,2) = **(10, 5, 7)** |

All finished → **SAFE**. Safe sequence: **< P1, P3, P4, P0, P2 >**. (Final Work equals the total — good sanity check.)

**Request example:** P1 requests (1, 0, 2).
- (1,0,2) ≤ Need P1 (1,2,2) ✔ and ≤ Available (3,3,2) ✔.
- Pretend: Available = (2,3,0), Allocation P1 = (3,0,2), Need P1 = (0,2,0).
- Safety: P1 (0,2,0) ≤ (2,3,0) ✔ → (5,3,2); P3 ✔ → (7,4,3); P4 ✔ → (7,4,5); P0 ✔ → (7,5,5); P2 ✔ → (10,5,7). **Safe → grant.**

```cpp
// bankers.cpp — Banker's algorithm: safety check + resource request
#include <iostream>
#include <vector>
using namespace std;
using Matrix = vector<vector<int>>;

// Returns true if safe; fills 'order' with a safe sequence.
bool isSafe(const vector<int>& avail, const Matrix& alloc, const Matrix& need, vector<int>& order) {
    int n = alloc.size(), m = avail.size();
    vector<int> work = avail;
    vector<bool> finish(n, false);
    order.clear();

    bool progress = true;
    while (progress) {                       // keep scanning while someone can finish
        progress = false;
        for (int i = 0; i < n; i++) {
            if (finish[i]) continue;
            bool canRun = true;
            for (int j = 0; j < m; j++)
                if (need[i][j] > work[j]) { canRun = false; break; }
            if (canRun) {                    // pretend P_i finishes and returns its resources
                for (int j = 0; j < m; j++) work[j] += alloc[i][j];
                finish[i] = true;
                order.push_back(i);
                progress = true;
            }
        }
    }
    return (int)order.size() == n;           // safe only if everyone finished
}

// Try to grant 'req' to process p. Returns true if granted (state updated).
bool request(int p, const vector<int>& req, vector<int>& avail, Matrix& alloc, Matrix& need) {
    int m = avail.size();
    for (int j = 0; j < m; j++) {
        if (req[j] > need[p][j]) { cout << "Error: more than declared max\n"; return false; }
        if (req[j] > avail[j])   { cout << "Must wait: not enough available\n"; return false; }
    }
    for (int j = 0; j < m; j++) { avail[j] -= req[j]; alloc[p][j] += req[j]; need[p][j] -= req[j]; }
    vector<int> order;
    if (isSafe(avail, alloc, need, order)) return true;
    for (int j = 0; j < m; j++) { avail[j] += req[j]; alloc[p][j] -= req[j]; need[p][j] += req[j]; } // undo
    return false;
}

int main() {
    Matrix alloc = {{0,1,0},{2,0,0},{3,0,2},{2,1,1},{0,0,2}};
    Matrix maxm  = {{7,5,3},{3,2,2},{9,0,2},{2,2,2},{4,3,3}};
    vector<int> avail = {3,3,2};

    int n = alloc.size(), m = avail.size();
    Matrix need(n, vector<int>(m));
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++) need[i][j] = maxm[i][j] - alloc[i][j];

    vector<int> order;
    if (isSafe(avail, alloc, need, order)) {
        cout << "SAFE. Sequence: ";
        for (int p : order) cout << "P" << p << " ";      // P1 P3 P4 P0 P2
        cout << "\n";
    } else cout << "UNSAFE\n";

    cout << "P1 requests (1,0,2): "
         << (request(1, {1,0,2}, avail, alloc, need) ? "GRANTED" : "DENIED") << "\n";
}
```

**Trap:** Banker's algorithm is rarely used in real OSes — processes don't know their max need in advance and the number of processes changes. Say: "It's a theoretical avoidance method; real systems use prevention (lock ordering) or detection (databases)."

### 7.8 Detection and Recovery

**Detection**
- Single instance per resource: build the **wait-for graph**, look for a **cycle** (DFS, O(V+E)).
- Multiple instances: a Banker's-like detection algorithm using current **Request** instead of Need.
- When to run: periodically, or when CPU usage drops strangely low. PostgreSQL runs a check after a lock wait exceeds `deadlock_timeout` (default 1 s).

**Recovery**
- **Kill processes**: kill all deadlocked ones, or kill one at a time until the cycle breaks. Choose the **victim** by: lowest priority, least work done, fewest resources held.
- **Resource preemption**: take resources from a victim and **roll it back** to a safe point.
- Watch for **starvation**: the same victim may be chosen every time → count rollbacks in the cost.

Databases do exactly this: abort one transaction with a "deadlock detected" error; the application should **retry**.

### 7.9 Deadlock vs Livelock vs Starvation ⭐

| | Deadlock | Livelock | Starvation |
|---|---|---|---|
| Are processes running? | No — all blocked | **Yes** — busy, changing state | Others run; the victim waits |
| Progress? | None | None | Others progress; one never does |
| Analogy | Two cars stuck on one-lane bridge | Two people in a corridor, both step aside to the **same** side again and again | A low-priority patient never called because emergencies keep arriving |
| Fix | Lock ordering, timeouts, detection | **Randomness / jitter** in retry | **Aging**, fair queues |

🔗 **Project link:** "Two kinds of retry fixes in LocalArena: DB deadlocks are retried with **jittered** backoff, so retries don't happen in lockstep (that would be a livelock). And when I **cut off lagging clients**, it caused a **livelock-like** state — they could never prove they had caught up — so I **throttled instead of disconnecting**."

---

## 8. Memory Management

### 8.1 Logical vs Physical Address

**Simple explanation**
- **Logical (virtual) address**: the address your program **thinks** it uses. Like a **flat number in a building** ("Flat 302").
- **Physical address**: the real location in RAM chips. Like the **GPS coordinates** of that flat.
- The **MMU** (Memory Management Unit, hardware in the CPU) translates logical → physical on every memory access.

**Key points**
- Every process can use address `0x1000`, and they all map to **different** physical places → isolation.
- Simplest MMU: **base (relocation) register + limit register**. `physical = base + logical`, and if `logical ≥ limit` → trap (segfault).
- **Address binding** can happen at compile time, load time, or **execution time** (modern: execution time, needs MMU).

### 8.2 Contiguous Allocation — First / Best / Worst Fit

**Simple explanation**
Each process gets **one continuous block** of memory. Free blocks are called **holes**. When a process comes, pick a hole:

| Strategy | Rule | Speed | Effect |
|---|---|---|---|
| **First fit** | First hole big enough | Fastest | Good in practice |
| **Best fit** | **Smallest** hole big enough | Slow (search all) | Leaves tiny useless holes |
| **Worst fit** | **Largest** hole | Slow (search all) | Leaves big holes, but big jobs may fail later |
| Next fit | Like first fit, but starts from where last search ended | Fast | Spreads allocation |

**Worked example** — Holes (in order): **100, 500, 200, 300, 600 KB**. Processes arrive: **212, 417, 112, 426 KB**. The leftover part of a hole stays as a smaller hole.

| Process | First fit | Best fit | Worst fit |
|---|---|---|---|
| 212 | 500 → left 288 | 300 → left 88 | 600 → left 388 |
| 417 | 600 → left 183 | 500 → left 83 | 500 → left 83 |
| 112 | 288 → left 176 | 200 → left 88 | 388 → left 276 |
| 426 | **must wait** (no hole ≥ 426) | 600 → left 174 | **must wait** |

Here **best fit** placed everyone. (In general, first fit and best fit are both better than worst fit; first fit is faster.)

### 8.3 Fragmentation ⭐

**Simple explanation**
- **Internal fragmentation**: wasted space **inside** an allocated block. You asked for 18 KB, got a 20 KB block → 2 KB wasted inside. Like booking a 4-seat table for 3 people.
- **External fragmentation**: total free memory is enough, but it is split into **small pieces that are not together**. Like 5 free single seats scattered in a cinema — a family of 5 cannot sit together.

| | Internal | External |
|---|---|---|
| Where | Inside allocated block | Between allocated blocks |
| Caused by | Fixed-size blocks (paging, fixed partitions) | Variable-size blocks (contiguous, segmentation) |
| Fix | Smaller block sizes | **Compaction** (move processes together — costly), **paging** |

**50-percent rule** (first fit): for N allocated blocks, about **0.5 N** blocks are lost to external fragmentation → one-third of memory may be unusable.

### 8.4 Paging ⭐

**Simple explanation**
Cut the process's logical memory into fixed-size pieces called **pages**, and cut RAM into same-size pieces called **frames** (typically **4 KB**). Any page can go into **any** free frame. A **page table** remembers which page is in which frame. Like a **book** whose pages are kept on random shelves, with an **index** telling where each page is.

**Key points**
- **No external fragmentation** (every frame fits every page).
- **Internal fragmentation** only in the last page (on average half a page per process).
- Logical address = **(page number p, offset d)**. Physical = **(frame number f, offset d)**. Offset doesn't change.

```
 Logical address (32 bits, 4 KB pages):
 +-------------------------+--------------+
 |  page number p (20 bits)| offset d (12)|
 +-------------------------+--------------+
            |                      |
            v                      |
      Page table[p] = f            |
            |                      |
            v                      v
 +-------------------------+--------------+
 |  frame number f         | offset d     |   = physical address
 +-------------------------+--------------+
```

**Calculation example**
- Page size 4 KB = 2¹² → offset = **12 bits**.
- 32-bit address → page number = 32 − 12 = **20 bits** → 2²⁰ ≈ **1 million pages**.
- Page table entry 4 bytes → page table size = 2²⁰ × 4 B = **4 MB per process**. Too big → **multi-level paging** (8.6).
- Example: logical address 8200, page size 4096 → page = 8200 / 4096 = **2**, offset = 8200 % 4096 = **8**. If page 2 is in frame 5 → physical = 5 × 4096 + 8 = **20488**.

**Page table entry (PTE) bits**: frame number, **valid/present** bit, **dirty** bit (modified?), **referenced** bit (used recently?), protection bits (read/write/execute), user/kernel bit.

### 8.5 TLB (Translation Lookaside Buffer) ⭐

**Simple explanation**
Without help, every memory access needs **2** memory reads: one for the page table, one for the data. The **TLB** is a tiny, very fast **cache of recent page → frame translations** inside the CPU. Like keeping your **most-called contacts on speed dial** instead of searching the phonebook.

**Key points**
- Small (tens to a few thousand entries), associative (searches all entries at once).
- **TLB hit**: translation found → 1 memory access. **TLB miss**: walk the page table, then store in TLB.
- Hit ratio is usually > 99% because of **locality of reference** (programs reuse nearby addresses).
- On a process context switch, TLB entries must be flushed or tagged with **ASID** (address-space ID). This is part of why process switches cost more than thread switches.

**Effective Access Time (EAT) formula**
```
EAT = h × (TLB time + mem time) + (1 − h) × (TLB time + 2 × mem time)
```
Example: TLB = 20 ns, memory = 100 ns, hit ratio h = 0.8
→ EAT = 0.8 × 120 + 0.2 × 220 = 96 + 44 = **140 ns** (vs 200 ns with no TLB).

(For k-level paging, a miss costs `TLB + (k + 1) × mem`.)

### 8.6 Multi-level Paging, Hashed & Inverted Page Tables

**Simple explanation**
A 4 MB page table per process is wasteful, since most of the address space is unused. **Multi-level paging** = **page the page table**. Like a **book index split into chapters**: first find the chapter (outer table), then the page inside the chapter (inner table). Inner tables for unused regions are simply not created.

```
 32-bit address with two levels:  | p1 (10 bits) | p2 (10 bits) | offset (12 bits) |
   outer table[p1] -> inner table ; inner table[p2] -> frame ; frame + offset
```
- x86-64 uses **4-level** paging (48-bit addresses), newer CPUs support 5-level.
- Cost: more memory accesses on a TLB miss → TLB is even more important.
- **Hashed page table**: hash of page number → chain of entries (used for large address spaces).
- **Inverted page table**: **one entry per physical frame** (not per page) → small table, but lookup needs search/hash, and sharing memory is harder.

**Huge pages** (2 MB / 1 GB): one TLB entry covers much more memory → fewer TLB misses. Databases and JVMs often use them.

### 8.7 Segmentation

**Simple explanation**
Divide memory by **meaning**, not fixed size: code segment, data segment, stack segment, heap. Like a **book divided into chapters** of different lengths. Address = **(segment number, offset)**. Segment table has **base** and **limit** for each segment.

| | Paging | Segmentation |
|---|---|---|
| Unit size | Fixed (pages) | Variable (segments) |
| Visible to programmer? | No | Yes (logical units) |
| Fragmentation | Internal | **External** |
| Table | Page table (page → frame) | Segment table (base, limit) |
| Sharing/protection | Per page | Natural per segment (e.g. code read-only) |

- **Segmentation with paging**: each segment is itself paged (old x86 / Intel). Modern 64-bit Linux mostly uses **flat** segments and relies on paging.

### 8.8 Stack vs Heap

| | Stack | Heap |
|---|---|---|
| What | Function frames, local variables, return addresses | `new`/`malloc`, objects that live beyond a function |
| Per | **Each thread** has its own | Shared by all threads of a process |
| Managed | Automatically (push/pop) | Manually (`delete`/`free`) or by **GC** |
| Speed | Very fast (just move stack pointer) | Slower (allocator search, possible locks) |
| Size | Small (Linux default ~8 MB main thread) | Large |
| Errors | **Stack overflow** (deep recursion) | **Memory leak**, fragmentation, use-after-free |

---

## 9. Virtual Memory and Page Replacement

### 9.1 Virtual Memory ⭐

**Simple explanation**
**Virtual memory** gives every process the illusion of a **huge, private, continuous** memory, even if RAM is small. Only the pages currently needed are in RAM; the rest wait on disk (**swap space**). Like a **library**: you keep only the few books you are reading on your desk (RAM); the rest stay on the shelves (disk), and you fetch one when needed.

**Benefits**
- Programs can be **larger than RAM**.
- More programs fit in memory → higher multiprogramming.
- **Isolation** and protection between processes.
- Easy **sharing** (shared libraries, shared memory) and **copy-on-write** after fork.
- Programmer doesn't care where things physically live.

🔗 **Project link (impress line):** "vLLM's **PagedAttention** applies exactly this idea to the LLM **KV cache** — fixed-size **blocks** + a **block table** instead of one large contiguous allocation per request. That cut KV memory waste from **60–80% to under 4%** — the same reason OSes moved from contiguous allocation to paging: no external fragmentation. It also uses **copy-on-write** for shared prompt prefixes."

### 9.2 Demand Paging

**Simple explanation**
Load a page **only when it is first used**, not at program start. Like a **lazy** student who opens a chapter only when the exam asks about it. A page that is not loaded has its **valid bit = 0** in the page table.

- **Pure demand paging**: start with zero pages in memory; every first touch faults.
- Works well because of **locality of reference**: at any moment a program uses a small set of pages.

### 9.3 Page Fault — Steps ⭐

**Simple explanation**
A **page fault** happens when a program touches a page that is **not in RAM**. It is not an error — it is the OS's signal to "go fetch it."

**Steps**
1. CPU accesses an address; MMU finds the PTE **valid bit = 0** → raises a **page-fault trap** to the kernel.
2. OS checks: is this address legal for the process? If **illegal** → kill the process (**segmentation fault**).
3. If legal but not in memory: find a **free frame**. If none, run **page replacement** to pick a victim (if victim is **dirty**, write it back to disk first).
4. Schedule a **disk read** of the needed page into the frame. (Process is blocked; CPU runs someone else.)
5. When the read finishes, **update the page table** (frame number, valid = 1).
6. **Restart the instruction** that caused the fault. The process continues as if nothing happened.

**Minor vs major fault**: *minor* = page is already in RAM somewhere (e.g. shared library, page cache), only the mapping is missing — fast. *Major* = must read from disk — slow (milliseconds).

**Cost example (EAT with page faults)**
Memory access = 200 ns, page fault service = 8 ms, fault rate p = 1/1000:
`EAT = (1 − p) × 200 + p × 8,000,000 ≈ 200 + 8,000 = 8,200 ns` → **40× slower**. Even one fault per thousand accesses is a disaster. That is why page-fault rate must be tiny.

### 9.4 Page Replacement Algorithms ⭐

When RAM is full and a new page is needed, which page do we **evict**?

| Algorithm | Evicts | Notes |
|---|---|---|
| **FIFO** | The page that came in **first** (oldest) | Simple; suffers **Belady's anomaly** |
| **Optimal (OPT / MIN)** | The page that will be used **farthest in the future** | Best possible; **impossible** in practice (needs the future). Used as a benchmark |
| **LRU** (Least Recently Used) | The page **not used for the longest time** | Close to optimal; uses the past to guess the future. Exact LRU is costly in hardware |
| **Clock / Second-chance** | FIFO, but skip pages whose **reference bit = 1** (clear it, give a second chance) | Cheap **approximation of LRU** — what real OSes use |
| LFU / MFU | Least / most frequently used | Rarely good in practice |

#### Worked example — 3 frames

Reference string: **7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1** (20 references)
F = page fault, H = hit. Frames listed as a set.

| # | Ref | FIFO frames | | LRU frames | | Optimal frames | |
|---|---|---|---|---|---|---|---|
| 1 | 7 | 7 | F | 7 | F | 7 | F |
| 2 | 0 | 7 0 | F | 7 0 | F | 7 0 | F |
| 3 | 1 | 7 0 1 | F | 7 0 1 | F | 7 0 1 | F |
| 4 | 2 | 0 1 2 (out 7) | F | 0 1 2 (out 7) | F | 2 0 1 (out 7) | F |
| 5 | 0 | 0 1 2 | H | 0 1 2 | H | 2 0 1 | H |
| 6 | 3 | 1 2 3 (out 0) | F | 0 2 3 (out 1) | F | 2 0 3 (out 1) | F |
| 7 | 0 | 2 3 0 (out 1) | F | 0 2 3 | H | 2 0 3 | H |
| 8 | 4 | 3 0 4 (out 2) | F | 0 3 4 (out 2) | F | 2 4 3 (out 0) | F |
| 9 | 2 | 0 4 2 (out 3) | F | 0 4 2 (out 3) | F | 2 4 3 | H |
| 10 | 3 | 4 2 3 (out 0) | F | 4 2 3 (out 0) | F | 2 4 3 | H |
| 11 | 0 | 2 3 0 (out 4) | F | 2 3 0 (out 4) | F | 2 0 3 (out 4) | F |
| 12 | 3 | 2 3 0 | H | 2 3 0 | H | 2 0 3 | H |
| 13 | 2 | 2 3 0 | H | 2 3 0 | H | 2 0 3 | H |
| 14 | 1 | 3 0 1 (out 2) | F | 3 2 1 (out 0) | F | 2 0 1 (out 3) | F |
| 15 | 2 | 0 1 2 (out 3) | F | 3 2 1 | H | 2 0 1 | H |
| 16 | 0 | 0 1 2 | H | 2 1 0 (out 3) | F | 2 0 1 | H |
| 17 | 1 | 0 1 2 | H | 2 1 0 | H | 2 0 1 | H |
| 18 | 7 | 1 2 7 (out 0) | F | 1 0 7 (out 2) | F | 7 0 1 (out 2) | F |
| 19 | 0 | 2 7 0 (out 1) | F | 1 0 7 | H | 7 0 1 | H |
| 20 | 1 | 7 0 1 (out 2) | F | 1 0 7 | H | 7 0 1 | H |
| | **Total faults** | **15** | | **12** | | **9** | |

How to decide quickly:
- **FIFO**: evict the one that **entered** earliest (keep a queue).
- **LRU**: look **backwards** — evict the one whose **last use** is farthest back.
- **Optimal**: look **forwards** — evict the one whose **next use** is farthest away (or never used again).

Hit ratio for LRU = 8/20 = 40%. Fault ratio = 12/20 = 60%.

### 9.5 Belady's Anomaly ⭐

**Simple explanation**
Normally more frames → fewer faults. **Belady's anomaly**: with **FIFO**, giving **more frames** can cause **more** page faults.

Classic string: **1 2 3 4 1 2 5 1 2 3 4 5**
- FIFO with **3 frames → 9 faults**.
- FIFO with **4 frames → 10 faults**. ← more frames, more faults!

**Key points**
- LRU and Optimal **never** show Belady's anomaly. They are **stack algorithms**: the set of pages in memory with n frames is always a **subset** of the set with n+1 frames.
- FIFO is not a stack algorithm.

### 9.6 Code: FIFO, LRU, Optimal Page-Fault Counter

```cpp
// page_faults.cpp — counts page faults for FIFO, LRU and Optimal
#include <iostream>
#include <list>
#include <queue>
#include <unordered_map>
#include <unordered_set>
#include <vector>
using namespace std;

int fifoFaults(const vector<int>& refs, int frames) {
    unordered_set<int> inMem;      // pages currently in RAM
    queue<int> order;              // arrival order (front = oldest)
    int faults = 0;
    for (int page : refs) {
        if (inMem.count(page)) continue;            // hit
        faults++;
        if ((int)inMem.size() == frames) {          // full: evict oldest
            inMem.erase(order.front());
            order.pop();
        }
        inMem.insert(page);
        order.push(page);
    }
    return faults;
}

int lruFaults(const vector<int>& refs, int frames) {
    list<int> recent;                               // front = most recent, back = least recent
    unordered_map<int, list<int>::iterator> pos;    // page -> its node in 'recent'
    int faults = 0;
    for (int page : refs) {
        auto it = pos.find(page);
        if (it != pos.end()) {                      // hit: move to front
            recent.erase(it->second);
        } else {
            faults++;
            if ((int)recent.size() == frames) {     // full: evict least recent (back)
                pos.erase(recent.back());
                recent.pop_back();
            }
        }
        recent.push_front(page);
        pos[page] = recent.begin();
    }
    return faults;
}

int optimalFaults(const vector<int>& refs, int frames) {
    vector<int> mem;
    int faults = 0;
    for (int i = 0; i < (int)refs.size(); i++) {
        bool hit = false;
        for (int p : mem) if (p == refs[i]) { hit = true; break; }
        if (hit) continue;
        faults++;
        if ((int)mem.size() < frames) { mem.push_back(refs[i]); continue; }
        // evict the page whose next use is farthest away (or never)
        int victim = 0, farthest = -1;
        for (int k = 0; k < (int)mem.size(); k++) {
            int next = INT32_MAX;
            for (int j = i + 1; j < (int)refs.size(); j++)
                if (refs[j] == mem[k]) { next = j; break; }
            if (next > farthest) { farthest = next; victim = k; }
        }
        mem[victim] = refs[i];
    }
    return faults;
}

int main() {
    vector<int> refs = {7,0,1,2,0,3,0,4,2,3,0,3,2,1,2,0,1,7,0,1};
    cout << "FIFO = " << fifoFaults(refs, 3)        // 15
         << ", LRU = " << lruFaults(refs, 3)        // 12
         << ", OPT = " << optimalFaults(refs, 3) << "\n";   // 9

    vector<int> belady = {1,2,3,4,1,2,5,1,2,3,4,5};
    cout << "Belady FIFO: 3 frames = " << fifoFaults(belady, 3)    // 9
         << ", 4 frames = " << fifoFaults(belady, 4) << "\n";      // 10
}
```

### 9.7 LRU Cache — HashMap + Doubly Linked List, O(1) ⭐ (very common coding question)

**Simple explanation**
Same idea as LRU page replacement, but for a key–value cache (LeetCode 146). We need `get` and `put` in **O(1)**.
- **Hash map**: key → node, so we **find** any key in O(1).
- **Doubly linked list**: order of use. **Front = most recent, back = least recent.** Moving or removing a node is O(1) because each node knows its `prev` and `next`.
- Two **dummy (sentinel) nodes** `head` and `tail` remove all "is the list empty?" special cases.

```
 head <-> [k3] <-> [k1] <-> [k7] <-> tail
          most                least
          recent              recent (evict this)
```

```cpp
// lru_cache.cpp — O(1) get/put with unordered_map + hand-written doubly linked list
#include <iostream>
#include <unordered_map>

class LRUCache {
    struct Node {
        int key, val;
        Node *prev = nullptr, *next = nullptr;
        Node(int k, int v) : key(k), val(v) {}
    };

    int cap;
    std::unordered_map<int, Node*> mp;   // key -> node
    Node* head;                          // dummy: head->next is MOST recent
    Node* tail;                          // dummy: tail->prev is LEAST recent

    void unlink(Node* n) {               // remove node from list, O(1)
        n->prev->next = n->next;
        n->next->prev = n->prev;
    }
    void addFront(Node* n) {             // insert right after head, O(1)
        n->next = head->next;
        n->prev = head;
        head->next->prev = n;
        head->next = n;
    }

public:
    explicit LRUCache(int capacity) : cap(capacity) {
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head->next = tail;
        tail->prev = head;
    }
    ~LRUCache() {                        // free all nodes
        Node* cur = head;
        while (cur) { Node* nx = cur->next; delete cur; cur = nx; }
    }
    LRUCache(const LRUCache&) = delete;              // no copies (raw pointers inside)
    LRUCache& operator=(const LRUCache&) = delete;

    int get(int key) {
        auto it = mp.find(key);
        if (it == mp.end()) return -1;   // miss
        Node* n = it->second;
        unlink(n);                       // mark as most recently used
        addFront(n);
        return n->val;
    }

    void put(int key, int value) {
        auto it = mp.find(key);
        if (it != mp.end()) {            // key exists: update + move to front
            Node* n = it->second;
            n->val = value;
            unlink(n);
            addFront(n);
            return;
        }
        if ((int)mp.size() == cap) {     // full: evict least recent
            Node* lru = tail->prev;
            unlink(lru);
            mp.erase(lru->key);          // this is why the node stores its key
            delete lru;
        }
        Node* n = new Node(key, value);
        addFront(n);
        mp[key] = n;
    }
};

int main() {
    LRUCache c(2);
    c.put(1, 1);
    c.put(2, 2);
    std::cout << c.get(1) << "\n";   // 1   (order now: 1, 2)
    c.put(3, 3);                     // evicts key 2
    std::cout << c.get(2) << "\n";   // -1
    c.put(4, 4);                     // evicts key 1
    std::cout << c.get(1) << "\n";   // -1
    std::cout << c.get(3) << "\n";   // 3
    std::cout << c.get(4) << "\n";   // 4
}
```

**Interview tips**
- First explain **why** each structure: map alone can't tell order; list alone can't find in O(1).
- Why does the node store the **key**? So when we evict the tail node, we can erase it from the map.
- Shorter version: `std::list<pair<int,int>>` + `unordered_map<int, list<...>::iterator>` and `list.splice()` to move to front. Ask the interviewer if STL list is allowed.
- Follow-ups: **thread-safe** LRU (wrap with a mutex; or shard the cache into N parts each with its own lock); **LFU cache** (LeetCode 460); Redis uses **approximate LRU** (samples a few keys) to save memory.

### 9.8 Thrashing ⭐

**Simple explanation**
**Thrashing** = the system spends more time **swapping pages in and out** than doing real work. Like a student with a tiny desk who keeps returning one book to the shelf to fetch another, and then needs the first one back — no studying gets done.

**Why it happens (the vicious cycle)**
1. Too many processes, each has too few frames.
2. Page faults rise → processes wait for disk → **CPU utilization drops**.
3. The OS sees low CPU use and thinks "let's add more processes" (long-term scheduler).
4. Even fewer frames each → even more faults → collapse.

```
 CPU utilization
   ^        ____
   |      /      \
   |    /          \   <- thrashing starts
   |  /              \___
   +------------------------> degree of multiprogramming
```

**Fixes**
- **Working-set model**: the working set = pages used in the last Δ references. Keep each process's working set in RAM; if the sum > total frames, **suspend** a process.
- **Page-fault frequency (PFF)**: if a process's fault rate is too high → give it more frames; too low → take some away.
- Reduce degree of multiprogramming; add RAM; use local (not global) replacement.

**Backend link:** In containers, memory limits that are too tight cause heavy GC or swapping, and then **p99 latency explodes** — the same pattern.

### 9.9 Copy-on-Write (COW) ⭐

**Simple explanation**
After `fork()`, parent and child **share the same physical pages**, marked **read-only**. Only when one of them **writes** to a page does the OS make a **private copy** of that one page. Like two students sharing one textbook; only when one wants to **write notes** in it do they photocopy that page.

**Key points**
- Makes `fork()` fast — no copying of the full memory up front.
- `fork()` + immediate `exec()` (the shell pattern) copies almost nothing.
- Used in: Redis background save (`BGSAVE` forks and the child writes a snapshot while the parent keeps serving), file systems (ZFS, Btrfs), container image layers.
- `vfork()`: older trick — child borrows the parent's memory and the parent is paused until `exec`.

🔗 **Project link:** "vLLM also uses **copy-on-write** blocks so multiple sequences can share the same prompt prefix KV blocks until they diverge."

---

## 10. Storage and File Systems

### 10.1 File System Basics

**Simple explanation**
A **file system** is the **library catalog** for the disk. The disk itself only stores numbered blocks. The file system turns those blocks into named files and folders, and remembers which blocks belong to which file.

**Key points**
- A **file** = named collection of bytes + metadata (size, owner, permissions, timestamps).
- A **directory** = a special file that maps **names → inode numbers**.
- Common file systems: **ext4** (Linux default), XFS, Btrfs, NTFS (Windows), APFS (macOS), FAT32 (USB drives).
- **Journaling** (ext4, NTFS): before changing the file system, write the plan to a **journal** (log). After a crash, replay the log → no corruption. Same idea as a database **write-ahead log (WAL)**.
- **File descriptor (fd)**: a small integer a process uses to refer to an open file/socket/pipe. 0 = stdin, 1 = stdout, 2 = stderr. "Too many open files" error = process hit its fd limit (`ulimit -n`) — common in servers with many connections.

**Allocation methods**
| Method | Idea | Good | Bad |
|---|---|---|---|
| Contiguous | File in consecutive blocks | Fast sequential + random access | External fragmentation, hard to grow |
| Linked | Each block points to next | No external fragmentation | Slow random access; a broken pointer loses the rest (FAT fixes this with a table) |
| Indexed | An **index block** lists all block addresses | Fast random access, no fragmentation | Index overhead |

### 10.2 Inode ⭐

**Simple explanation**
An **inode** is a file's **ID card**. It stores **everything about the file except its name**: size, owner, permissions, timestamps, link count, and **pointers to data blocks**. The **name lives in the directory**, which maps name → inode number.

**Inode block pointers (ext2/ext3 style)**
```
 inode
 +-----------------------+
 | metadata              |
 | 12 direct pointers  --+--> data blocks
 | single indirect     --+--> block of pointers --> data blocks
 | double indirect     --+--> block of ptrs --> blocks of ptrs --> data
 | triple indirect     --+--> 3 levels --> data
 +-----------------------+
```
Small files are fast (direct pointers). Large files are still possible (indirect pointers). (ext4 uses **extents** — "start block + length" — instead, which is more efficient for large files.)

**Hard link vs Soft (symbolic) link**

| | Hard link | Soft link (symlink) |
|---|---|---|
| What | Another **name** pointing to the **same inode** | A small file containing the **path** of another file |
| If original deleted | Data still there (inode freed only when link count = 0) | Link becomes **dangling** (broken) |
| Across file systems | No | Yes |
| To directories | No (normally) | Yes |
| Command | `ln a b` | `ln -s a b` |

**Interview tip:** "Deleting a file" really means removing a name; data is freed when the link count reaches 0 **and** no process has it open. That's why a big log file deleted while a process still writes to it does **not** free disk space until the process closes it.

### 10.3 Disk Structure (HDD vs SSD)

- **HDD**: spinning platters, a moving arm. Access time = **seek time** (move arm to track — the biggest part) + **rotational latency** + transfer time. Random I/O is very slow (~10 ms), sequential is fast.
- **SSD**: flash memory, no moving parts, random reads ~100 µs or less. Writes happen in pages, erases in large blocks → needs **wear leveling** and **TRIM**. Disk scheduling matters much less for SSDs.

### 10.4 Disk Scheduling ⭐

**Goal:** reduce total **head movement** (seek distance) on an HDD.

| Algorithm | How it works | Notes |
|---|---|---|
| **FCFS** | Serve in order of arrival | Fair, but lots of movement |
| **SSTF** (Shortest Seek Time First) | Go to the **nearest** request | Less movement; far requests can **starve** |
| **SCAN** (elevator) | Move in one direction to the **end of the disk**, then reverse | Like a lift going up to the top floor, then down |
| **C-SCAN** (circular) | Move one direction to the end, **jump back to the start** without serving, continue same direction | More **uniform** waiting time |
| **LOOK** | Like SCAN, but turn around at the **last request**, not the disk end | Practical SCAN |
| **C-LOOK** | Like C-SCAN, but go only as far as the last request, then jump to the **lowest request** | Practical C-SCAN |

#### Worked example

Cylinders 0–199. Head at **53**, moving **towards higher numbers** (for SCAN-family).
Request queue: **98, 183, 37, 122, 14, 124, 65, 67**.
Sorted: 14, 37, 65, 67, 98, 122, 124, 183.

| Algorithm | Order of service | Total head movement |
|---|---|---|
| **FCFS** | 53 → 98 → 183 → 37 → 122 → 14 → 124 → 65 → 67 | 45+85+146+85+108+110+59+2 = **640** |
| **SSTF** | 53 → 65 → 67 → 37 → 14 → 98 → 122 → 124 → 183 | 12+2+30+23+84+24+2+59 = **236** |
| **SCAN** (up) | 53 → 65 → 67 → 98 → 122 → 124 → 183 → **199** → 37 → 14 | (199−53) + (199−14) = 146 + 185 = **331** |
| **C-SCAN** (up) | 53 → … → 183 → **199** → jump to **0** → 14 → 37 | 146 + 199 (jump) + 37 = **382** (or **183** if the jump is not counted) |
| **LOOK** (up) | 53 → 65 → 67 → 98 → 122 → 124 → 183 → 37 → 14 | (183−53) + (183−14) = 130 + 169 = **299** |
| **C-LOOK** (up) | 53 → … → 183 → jump to **14** → 37 | 130 + 169 (jump) + 23 = **322** (or **153** if the jump is not counted) |

(If the head was moving **down** instead: SCAN = 53 → 37 → 14 → 0 → 65 → … → 183 = 53 + 183 = **236**.)

**Trap:** Books differ on whether the C-SCAN/C-LOOK return jump is counted. **State your assumption** before calculating. Also: SSTF is **not optimal** in general — it is greedy and can starve.

### 10.5 RAID (brief)

**Simple explanation**
**RAID** = combine many disks to get more **speed**, more **safety**, or both.

| Level | Idea | Min disks | Survives | Usable space | Use |
|---|---|---|---|---|---|
| **RAID 0** | **Striping** — split data across disks | 2 | **0 disk failures** | 100% | Speed only (temp data) |
| **RAID 1** | **Mirroring** — same data on 2 disks | 2 | 1 disk | 50% | OS disks, small DBs |
| **RAID 5** | Striping + **distributed parity** | 3 | 1 disk | (n−1)/n | General file servers |
| **RAID 6** | Striping + **double parity** | 4 | 2 disks | (n−2)/n | Large arrays |
| **RAID 10** (1+0) | Mirror pairs, then stripe across them | 4 | 1 per mirror pair | 50% | **Databases** (fast + safe) |

**Parity** = XOR of the data blocks. If one block is lost, XOR the rest to rebuild it.
**Trap:** **RAID is not a backup.** Deleting a file or ransomware is copied to all disks instantly.

---

## 11. I/O Systems

### 11.1 Polling vs Interrupts ⭐

**Simple explanation**
- **Polling**: the CPU **keeps asking** the device "are you done? are you done?" Like checking your phone every 10 seconds for a delivery message.
- **Interrupt**: the device **tells** the CPU when it's done by raising a signal. Like the delivery person **ringing the doorbell**.

| | Polling | Interrupts |
|---|---|---|
| CPU use | Wastes CPU if device is slow | CPU does other work meanwhile |
| Latency | Can be lowest for very fast devices | Small overhead per interrupt |
| Best for | Very fast / very busy devices (high-speed NICs, NVMe under heavy load — e.g. Linux NAPI switches to polling under load) | Slow or occasional events (keyboard, disk) |

**Interrupt handling steps:** device raises interrupt → CPU finishes current instruction → saves state → jumps to the **interrupt handler** via the **interrupt vector table** → handler does minimal work (top half) and defers the rest (bottom half / softirq) → restore state, continue.

**Types:** hardware interrupt (device), software interrupt / **trap** (system call, exception like divide-by-zero or page fault).

### 11.2 DMA (Direct Memory Access)

**Simple explanation**
Without DMA, the CPU copies every byte from the device to memory itself — like a manager carrying every box personally. With **DMA**, a helper (the **DMA controller**) moves the whole block between device and RAM, and **interrupts the CPU only once** when the transfer is done.

- CPU sets up: source, destination, size → DMA does the copy → one interrupt at the end.
- Frees the CPU for real work. Used by disks, network cards, GPUs.
- **Cycle stealing**: DMA occasionally takes the memory bus, slightly slowing the CPU.

### 11.3 Blocking vs Non-blocking vs Asynchronous I/O ⭐

**Simple explanation (restaurant analogy)**
- **Blocking**: you order food and **stand at the counter** until it's ready. You can't do anything else.
- **Non-blocking**: you ask "is it ready?" It says "no" **immediately** (`EAGAIN`). You go do something else and ask again later.
- **I/O multiplexing (select/poll/epoll)**: you give the counter a list of all your orders and **wait for a buzzer** that says *which* order is ready.
- **Asynchronous I/O**: you order and leave; the food is **delivered to your table** and you're notified when it's **done** (Linux `io_uring`, Windows IOCP).

| Model | Thread blocks? | Who copies data? | Notified when |
|---|---|---|---|
| Blocking | Yes, until data is ready and copied | Kernel, during the call | — |
| Non-blocking | No, returns `EAGAIN` | Kernel, when you retry | You poll |
| Multiplexing (epoll) | Blocks in `epoll_wait` on **many** fds at once | You call `read` after | fd is **ready** |
| Async (io_uring) | No | Kernel, in background | Operation is **complete** |

**Trap:** "Non-blocking" and "asynchronous" are not the same. Non-blocking = "tell me now if you can't do it." Async = "do it and tell me when finished."

### 11.4 select vs poll vs epoll ⭐

**Simple explanation**
A server has 10,000 open connections. Most are idle. How does **one thread** find out which ones have data?

| | `select` | `poll` | `epoll` (Linux) |
|---|---|---|---|
| fd limit | **1024** (FD_SETSIZE) | No fixed limit | No fixed limit |
| Each call | Copy the whole fd set to kernel; kernel scans **all** fds | Same — pass array, kernel scans all | Register fds **once** (`epoll_ctl`); `epoll_wait` returns **only the ready ones** |
| Cost per call | **O(n)** | **O(n)** | **O(ready)** |
| Analogy | Teacher calls **every** student's name to see who has a question | Same, but with a longer list | Students with questions **raise their hand**; teacher sees only them |

Modes: **level-triggered** (default — keeps telling you while data remains) vs **edge-triggered** (tells you **once** when the state changes; you must read until `EAGAIN`).
macOS/BSD have **kqueue** (same idea).

**epoll loop sketch (Linux, pseudocode-level C)**
```
epfd = epoll_create1(0);
epoll_ctl(epfd, EPOLL_CTL_ADD, listen_fd, {EPOLLIN});
loop:
    n = epoll_wait(epfd, events, MAX, -1);       // sleep until some fds are ready
    for each ready fd in events[0..n):
        if fd == listen_fd: conn = accept(); set non-blocking; epoll_ctl(ADD, conn)
        else: read(fd) until EAGAIN; handle request; write response
```

This is the core of **Nginx, Redis, Node.js, Netty**, and **Go's netpoller**. It solves the **C10K problem** (10,000 concurrent connections) without 10,000 threads.

🔗 **Project link:** "That's how a single Go server holds huge numbers of WebSocket connections — goroutines **look** blocking, but the Go runtime parks them and uses **epoll** underneath. So in my TTL chat, 2 goroutines per connection is cheap."

### 11.5 Buffering, Caching, Spooling

- **Buffer**: temporary memory area for data moving between two places at different speeds (e.g. network card → app). 
- **Cache**: copy of frequently used data in faster storage (the **page cache** keeps recently read file blocks in RAM — why the second read of a file is fast).
- **Spooling**: put jobs in a queue on disk for a slow device (print spooler).
- **Zero-copy** (`sendfile`): send file data to a socket **without copying it through user space** — used by Kafka and Nginx.

### 11.6 Kernel Socket Buffers (backend gold)

**Simple explanation**
Every TCP socket has a **send buffer** and a **receive buffer** inside the kernel. When your app calls `write()`, data goes into the send buffer and `write()` returns — even if the client hasn't received anything yet. These buffers are a **hidden queue** under your application.

🔗 **Project link (rare, impressive):** "In LocalArena, **Linux TCP send/receive buffers absorbed hundreds of game snapshots**, so slow clients were **seconds behind** while my **application-level queue looked healthy**. Lesson: a bounded app queue doesn't bound **end-to-end latency** if kernel buffers underneath can grow. Then cutting off lagging clients caused a livelock-like state, so I **throttled instead of disconnecting**."

---

## 12. Backend-Relevant Extras

### 12.1 Context Switch Cost in Practice

- A thread context switch costs roughly **a few microseconds** directly, plus indirect costs (cold CPU caches, TLB misses) that can be larger.
- 10,000 threads → each has a stack (often ~1–8 MB reserved) + scheduling overhead → memory and CPU wasted.
- Ways backends avoid it: **thread pools**, **event loops** (epoll), **goroutines / virtual threads** (Java 21 virtual threads are the same M:N idea as goroutines), batching syscalls.
- Measure on Linux: `vmstat 1` (column `cs`), `pidstat -w`, `perf`.

### 12.2 Garbage Collection Basics and p99 Latency

**Simple explanation**
**Garbage collection (GC)** automatically frees heap memory that the program can no longer reach. Like a **cleaner** who removes plates nobody is using any more. C++ has no GC (you use RAII / smart pointers); Java, Go, Python have GC.

| Technique | Idea |
|---|---|
| **Reference counting** | Each object counts who points to it; free at 0. Simple, but **cannot free cycles** (Python adds a cycle detector; C++ `shared_ptr` is ref counting → use `weak_ptr` to break cycles) |
| **Mark and sweep** | Start from **roots** (stack, globals), **mark** everything reachable, **sweep** (free) the rest |
| **Generational** | "Most objects die young." Collect the **young** generation often and cheaply, the old one rarely (Java G1) |
| **Concurrent tri-color** | Mark while the program keeps running (white = unvisited, grey = to scan, black = done). Short **stop-the-world** pauses only (Go, Java ZGC) |

**Why it matters for backends:** a GC pause stops (or slows) request handling. Average latency looks fine, but **p99 / p99.9** (tail latency) jumps. Fixes: allocate less (reuse buffers, `sync.Pool` in Go), tune heap size (`GOGC`, `GOMEMLIMIT` in Go; heap and collector choice in Java), use low-pause collectors (ZGC, Shenandoah).

**p99** = 99% of requests are faster than this value. Interviewers love it because users feel the slow ones.

🔗 **Project link:** "I report **p50 / p95 / p99** in LocalArena — tick p99 **0.76 ms** (p50 0.07, p95 0.21) at 1,000 players — because averages hide pauses like GC."

### 12.3 Virtualization vs Containers ⭐

**Simple explanation**
- **Virtual Machine (VM)**: a **full fake computer** with its own OS kernel, running on a **hypervisor**. Like building **separate houses** on one plot — each has its own foundation, plumbing, everything.
- **Container**: a normal process on the **same host kernel**, but **isolated** so it thinks it's alone. Like **flats in one apartment building** — separate doors and walls, but they share the foundation (kernel).

```
     Virtual Machines                     Containers
 +-------+ +-------+ +-------+      +-------+ +-------+ +-------+
 | App A | | App B | | App C |      | App A | | App B | | App C |
 | Libs  | | Libs  | | Libs  |      | Libs  | | Libs  | | Libs  |
 |Guest OS|Guest OS|Guest OS |      +-------+-+-------+-+-------+
 +-------+-+-------+-+-------+      |   Container runtime        |
 |        Hypervisor          |      |   Host OS kernel (shared)  |
 |        Hardware            |      |   Hardware                 |
 +----------------------------+      +----------------------------+
```

| | VM | Container |
|---|---|---|
| Isolation | Strong (separate kernel) | Weaker (shared kernel) |
| Startup | Seconds to minutes | **Milliseconds** to seconds |
| Size | GBs | MBs |
| Overhead | Higher | Near-native |
| Run different OS kernel? | Yes (Windows VM on Linux host) | No (Linux containers need a Linux kernel) |
| Examples | KVM, VMware, Xen, AWS EC2 (Nitro) | Docker, containerd, Kubernetes pods |

**Hypervisor types**: **Type 1** (bare metal: KVM, Xen, ESXi) runs directly on hardware. **Type 2** (hosted: VirtualBox) runs on top of a normal OS.

**How containers work — two Linux kernel features**
1. **Namespaces** = **what you can see** (isolation). Each container gets its own view:
   - `pid` (own PID 1), `net` (own network interfaces, ports), `mnt` (own filesystem mounts), `uts` (own hostname), `ipc`, `user` (root inside ≠ root outside), `cgroup`.
2. **cgroups** (control groups) = **how much you can use** (limits): CPU, memory, I/O, number of processes.
   - Memory over the limit → the kernel's **OOM killer** kills the container (Kubernetes status `OOMKilled`).
   - CPU over the limit → **CPU throttling** → requests slow down → **p99 latency spikes** even though average CPU looks low.
3. Plus a layered image filesystem (**OverlayFS**, copy-on-write layers) and security filters (seccomp, capabilities).

Mnemonic: **Namespaces = walls. cgroups = budgets.**

🔗 **Project link:** "LocalArena's deployment experiments ran on a **kind** (Kubernetes-in-Docker) cluster — rolling, canary, blue-green and rollback at **600 req/s** with **HPA scaling 2 → 5 → 2 pods**, and **0 of 90,000** requests failed. Every pod there is just a Linux process inside namespaces with cgroup limits."

---

## 13. Most Asked OS Interview Questions (Top 40)

⭐ = top 15 — make sure you can say these without thinking.

**⭐ 1. Process vs thread?**
A process is a running program with its **own address space**. A thread is a unit of execution **inside** a process; threads share code, heap, globals and files, but each has its own stack and registers. Threads are cheaper to create and switch, but one crashing thread can kill the whole process. Example: Chrome uses processes per tab for safety; a web server uses a thread pool for speed.

**⭐ 2. What is a context switch and why is it expensive?**
Saving the CPU state (registers, program counter) of one process/thread into its PCB and loading another's. It's pure overhead: kernel entry, save/restore, and — the bigger cost — **cold caches and TLB** afterwards. Process switches also change page tables, so they cost more than thread switches. That's why event loops and goroutines exist.

**⭐ 3. Mutex vs semaphore?**
Mutex = a lock with **ownership**; one holder; only the locker unlocks (bathroom key). Semaphore = a **counter**; allows N holders (parking lot with N spots); no ownership, any thread can signal. Use mutex for mutual exclusion, counting semaphore to limit N concurrent users (e.g. connection pool), binary semaphore for signaling.

**⭐ 4. What is a deadlock? Necessary conditions?**
A set of processes each waiting for a resource held by another, so none can proceed. All four Coffman conditions must hold: **mutual exclusion, hold and wait, no preemption, circular wait**. Break any one to prevent it.

**⭐ 5. How do you prevent / handle deadlocks in practice?**
Prevention: **global lock ordering** (breaks circular wait), `std::scoped_lock` / try-lock with back-off, lock timeouts. Avoidance: Banker's algorithm (theory). Detection + recovery: databases build a wait-for graph and abort one transaction; the app **retries with jittered backoff**. 🔗 My LocalArena PostgreSQL deadlock was fixed by fixed-order row locking + conditional update + jittered retry.

**⭐ 6. Paging vs segmentation?**
Paging splits memory into **fixed-size** pages/frames → no external fragmentation, small internal fragmentation, invisible to the programmer. Segmentation splits by **logical units** (code, stack, heap) of variable size → natural protection/sharing, but external fragmentation. Modern systems mainly use paging.

**⭐ 7. What is virtual memory and why do we need it?**
Each process gets a large, private, contiguous **illusion** of memory; only needed pages are in RAM, the rest on disk. Benefits: programs larger than RAM, isolation between processes, more processes in memory, easy sharing and copy-on-write. The MMU + page tables + TLB translate virtual to physical addresses.

**⭐ 8. What is a page fault? What happens during it?**
Accessing a page not currently in RAM. MMU traps to the OS → OS checks the address is valid (else segfault) → finds a free frame (or evicts a victim, writing it back if dirty) → reads the page from disk → updates page table → **restarts the instruction**. Major faults (disk) cost milliseconds.

**⭐ 9. What is thrashing? How to fix it?**
When processes have too few frames, they page-fault constantly, the CPU sits idle waiting for disk, and the OS may add even more processes — throughput collapses. Fix: working-set model, page-fault-frequency control, reduce the number of processes, add RAM.

**⭐ 10. What is a race condition and critical section? What are the requirements of a solution?**
Race: the result depends on the timing of threads accessing shared data, with at least one write (`counter++` is load-add-store). Critical section: the code touching shared data. A solution must give **mutual exclusion, progress, and bounded waiting**.

**⭐ 11. Explain fork(). What does it return? How many processes after `fork()` 3 times?**
`fork()` creates a child copy of the process (copy-on-write). It returns **0 in the child**, the **child's PID in the parent**, −1 on error. After n forks in a row there are **2ⁿ** processes (so 8 for 3). `exec()` then replaces the child's program; `wait()` lets the parent collect the exit status.

**⭐ 12. Zombie vs orphan process?**
Zombie: child has exited but the parent hasn't called `wait()`, so its process-table entry (PID, exit code) remains. Orphan: parent exited first; the child is adopted by init/systemd (PID 1). Too many zombies exhaust PIDs; fix the parent (call `wait`, handle `SIGCHLD`).

**⭐ 13. Explain CPU scheduling algorithms. Which is best?**
FCFS (simple, convoy effect), SJF/SRTF (best average waiting time, needs burst prediction, starvation), Priority (starvation → aging), Round Robin (fair, good response time; quantum size matters), MLFQ (adapts to behavior). No single best: SRTF minimizes waiting time; RR is best for interactive response. Linux uses CFS / EEVDF.

**⭐ 14. Page replacement algorithms and Belady's anomaly?**
FIFO (evict oldest), Optimal (evict page used farthest in future — benchmark only), LRU (evict least recently used — close to optimal), Clock (cheap LRU approximation used in practice). Belady's anomaly: with **FIFO**, more frames can give **more** faults (1 2 3 4 1 2 5 1 2 3 4 5: 9 faults with 3 frames, 10 with 4). LRU and Optimal are stack algorithms and never show it.

**⭐ 15. Blocking vs non-blocking I/O; what is epoll?**
Blocking: the thread waits until data is ready. Non-blocking: returns immediately with `EAGAIN`. **epoll** lets one thread register thousands of sockets and get back **only the ready ones** (O(ready) vs O(n) for select/poll). It's the base of Nginx, Redis, Node.js and Go's netpoller, and solves C10K.

**16. User mode vs kernel mode? What is a system call?**
User mode can't run privileged instructions or touch hardware directly; kernel mode can. A system call (e.g. `read`, `write`, `fork`) is the controlled entry from user to kernel mode via a trap instruction; the kernel runs the handler and returns. It costs more than a function call, so we batch (buffered I/O).

**17. Monolithic vs microkernel?**
Monolithic: all OS services (drivers, FS, scheduler) inside the kernel — fast, but one bad driver can crash everything (Linux, with loadable modules). Microkernel: only minimal services in kernel; the rest run as user-space servers talking via IPC — safer, slower (MINIX, QNX). Hybrid: Windows NT, macOS XNU.

**18. What is a PCB?**
Process Control Block — the kernel's record of a process: PID, state, program counter, registers, scheduling info, memory info (page table), open files, accounting. Saved and loaded on every context switch. Linux: `task_struct`.

**19. Process states?**
New → Ready → Running → (Waiting on I/O → Ready) → Terminated. Running → Ready on preemption (time slice over). A waiting process always goes back to Ready, not directly to Running.

**20. IPC mechanisms? Which is fastest?**
Pipes, named pipes (FIFO), message queues, shared memory, sockets, signals, memory-mapped files. **Shared memory is fastest** (no kernel copy after setup) but needs synchronization. Sockets work across machines.

**21. Preemptive vs non-preemptive scheduling?**
Preemptive: the OS can take the CPU away (timer interrupt, higher-priority arrival) — RR, SRTF, CFS. Non-preemptive: a process keeps the CPU until it blocks or finishes — FCFS, SJF. Preemption gives better responsiveness but needs care with shared data.

**22. What is starvation and aging?**
Starvation: a process waits indefinitely because others keep getting priority (low priority in Priority scheduling, long jobs in SJF). Aging: gradually increase a waiting process's priority so it eventually runs.

**23. Spinlock vs mutex — when to use which?**
Spinlock busy-waits (burns CPU), good only for very short critical sections on multi-core, or in kernel code that can't sleep. Mutex puts waiters to sleep — better when holding time is longer. Linux futexes spin briefly then sleep.

**24. What is a semaphore? Explain wait and signal.**
An integer with atomic `wait/P` (if value > 0 decrement, else block) and `signal/V` (increment and wake a waiter). Counting semaphores limit access to N resources; binary semaphores (0/1) are used for signaling.

**25. Explain Banker's algorithm / safe state.**
A deadlock-avoidance method: each process declares its max need. A state is **safe** if there's an order in which every process can get its remaining need from available + resources released by earlier finishers. Before granting a request, pretend to grant and run the safety check; grant only if still safe. O(n²m). Rarely used in real OSes.

**26. Deadlock vs livelock vs starvation?**
Deadlock: all blocked, nothing changes. Livelock: processes keep running and reacting to each other but make no progress (two people side-stepping in a corridor) — fix with random jitter. Starvation: others progress, one never does — fix with aging/fair queues.

**27. Internal vs external fragmentation?**
Internal: wasted space inside an allocated block (fixed-size allocation like paging). External: enough total free memory but split into non-contiguous holes (variable-size allocation). Fix external with compaction or paging.

**28. What is a TLB? Effective access time?**
A small, fast hardware cache of recent page→frame translations. Hit → one memory access; miss → page-table walk. `EAT = h(t + m) + (1−h)(t + 2m)`. Hit rates are usually > 99% thanks to locality.

**29. Why multi-level paging?**
A single-level table for a 32-bit space with 4 KB pages needs 2²⁰ entries (~4 MB per process), mostly unused. Multi-level paging pages the page table itself; inner tables are created only for used regions. x86-64 uses 4 levels. Cost: more memory accesses on a TLB miss.

**30. What is copy-on-write?**
After `fork()`, parent and child share pages marked read-only; a page is copied only when someone writes to it. Makes fork cheap (especially fork + exec). Used by Redis BGSAVE, Btrfs/ZFS, container image layers, and vLLM's shared prefix blocks.

**31. User-level vs kernel-level threads? Multithreading models?**
User threads are managed by a library (fast switches, but one blocking syscall blocks all; no true parallelism if mapped to one kernel thread). Kernel threads are scheduled by the OS. Models: many-to-one, one-to-one (Linux, Java, C++), many-to-many (Go goroutines on OS threads).

**32. What is a condition variable? Why wait in a loop?**
A queue where threads wait (releasing the lock atomically) until another thread signals that a condition may be true. Always re-check the condition in a `while` loop (or use the predicate overload) because of **spurious wakeups** and because another thread may consume the item first.

**33. Explain the producer–consumer problem and its solution.**
Producers add to a bounded buffer, consumers remove. Producers wait when full, consumers wait when empty, and buffer access must be mutually exclusive. Solution: semaphores `empty=N`, `full=0`, `mutex=1` (wait on empty/full **before** mutex), or mutex + two condition variables. In Go: a buffered channel.

**34. Explain the dining philosophers problem. How do you avoid deadlock?**
5 philosophers, 5 forks, each needs two forks; if all pick up the left fork, circular wait → deadlock. Fixes: pick the lower-numbered fork first (resource ordering), allow only 4 at the table, asymmetric picking (odd/even), or pick both atomically. Watch out for starvation too.

**35. Readers–writers problem?**
Many readers may read together; a writer needs exclusive access. Reader-preference can starve writers; writer-preference can starve readers. Implement with a read-write lock (`std::shared_mutex`, Go `sync.RWMutex`). Good when reads far outnumber writes.

**36. Interrupt vs polling? What is DMA?**
Interrupt: the device signals the CPU when ready (CPU free meanwhile). Polling: the CPU repeatedly checks the device (wasteful unless the device is very fast/busy). DMA: a controller copies a whole block between device and memory and interrupts the CPU once at the end.

**37. What is an inode? Hard link vs soft link?**
An inode stores a file's metadata and block pointers — everything except its name (names are in directories). A hard link is another name for the same inode (survives deleting the original; same filesystem only). A soft link stores a path (can cross filesystems; breaks if the target is deleted).

**38. Disk scheduling algorithms?**
FCFS, SSTF (nearest first, may starve), SCAN (elevator, to the end then reverse), C-SCAN (one direction, jump back — uniform wait), LOOK/C-LOOK (turn at the last request). Example with head 53 and queue 98,183,37,122,14,124,65,67: FCFS 640, SSTF 236, LOOK (up) 299 cylinders.

**39. Virtual machines vs containers? How do containers isolate?**
VMs run a full guest OS on a hypervisor — strong isolation, heavier. Containers are processes sharing the host kernel — lightweight, fast startup. Isolation via **namespaces** (what you see: pid, net, mnt, uts, ipc, user) and limits via **cgroups** (CPU, memory, I/O). Memory over limit → OOM-kill; CPU over limit → throttling → p99 spikes.

**40. What happens when you run `./a.out` in a shell?**
The shell `fork()`s; the child calls `execve("./a.out")`; the kernel checks permissions, reads the ELF header, sets up a new address space (text, data, BSS, heap, stack), maps the dynamic loader which loads shared libraries (lazily, via demand paging), and jumps to `_start` → `main`. The parent shell `wait()`s; when the program exits, the shell collects the status (`$?`).

---

## 14. Last-minute Revision

**Basics**
- OS = resource manager + abstraction + protection. Kernel types: **monolithic (Linux)**, micro (MINIX/QNX), hybrid (Windows/macOS).
- User mode vs kernel mode; **system call** = controlled trap into kernel (`read`, `write`, `fork`).

**Processes & threads**
- States: New → Ready ⇄ Running → Waiting → Ready → Terminated. **PCB** = ID card + bookmark.
- `fork()` → 0 in child, child PID in parent; n forks → 2ⁿ processes. `exec` replaces, `wait` reaps.
- **Zombie** = dead, not reaped. **Orphan** = parent dead, adopted by PID 1.
- IPC: pipe, FIFO, message queue, **shared memory (fastest)**, socket (across machines), signal. `SIGKILL`/`SIGSTOP` can't be caught.
- Thread = shares heap/code/files, own stack/registers. Models: M:1, **1:1 (Linux/Java/C++)**, **M:N (Go)**.
- Context switch = overhead; cold cache/TLB is the hidden cost; thread switch < process switch.

**Scheduling**
- `TAT = CT − AT`, `WT = TAT − BT`, `RT = first run − AT`.
- FCFS (convoy), SJF (optimal non-preemptive avg WT), SRTF (optimal overall), Priority (+aging), RR (response time, quantum trade-off), MLFQ (adapts). Linux: **CFS** (red-black tree on vruntime) → **EEVDF** in 6.6+.
- Worked example averages (WT): FCFS 4.00, SJF 3.25, **SRTF 2.25**, RR(q=2) 4.50.

**Synchronization**
- Race = timing-dependent result on shared write. CS needs **mutual exclusion, progress, bounded waiting**.
- Peterson: `flag[i]=true; turn=j; while(flag[j] && turn==j);` — 2 processes; needs memory barriers on real CPUs.
- **Mutex** (key, ownership) vs **semaphore** (counter, N holders, no ownership). Spinlock = busy wait, short sections on multicore.
- Condition variable: `wait` releases lock + sleeps; always **while**-loop (spurious wakeups).
- Producer–consumer: `empty=N, full=0, mutex=1`; wait on empty/full **before** mutex. Go buffered channel.
- Readers–writers: `shared_mutex`. Dining philosophers: lower-numbered fork first.

**Deadlock**
- **Mutual exclusion, Hold & wait, No preemption, Circular wait** ("My Head Needs Coffee").
- RAG: no cycle → safe; cycle + single instances → deadlock; multiple instances → maybe.
- Prevention (lock ordering!) / Avoidance (Banker's) / Detection + recovery (DBs, retry with jitter) / Ignore.
- Banker's: `Need = Max − Alloc`; find Need ≤ Work, add its Alloc to Work, repeat. Example sequence **P1, P3, P4, P0, P2**.
- Livelock = busy but no progress (fix: jitter). Starvation = one never runs (fix: aging).

**Memory**
- Logical → physical by **MMU**. First/best/worst fit. Internal vs external fragmentation.
- Paging: page ↔ frame, address = (page, offset); 4 KB → 12-bit offset. Page table 2²⁰ × 4 B = 4 MB → **multi-level**.
- **TLB** = translation cache; `EAT = h(t+m) + (1−h)(t+2m)`.
- Segmentation = variable logical units, external fragmentation.
- Virtual memory + demand paging; page fault: trap → validate → free frame/evict → disk read → update PTE → restart instruction.
- FIFO / LRU / OPT on 7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1 with 3 frames: **15 / 12 / 9**.
- **Belady** (FIFO only): 1 2 3 4 1 2 5 1 2 3 4 5 → 9 faults (3 frames), 10 (4 frames).
- **Thrashing**: too few frames → fault storm → CPU idle. Fix: working set, PFF, fewer processes.
- **COW**: share after fork, copy on first write.
- **LRU cache**: hash map + doubly linked list (front = recent, back = evict), sentinel head/tail, node stores key.

**Storage & I/O**
- Inode = metadata + block pointers, **no name**. Hard link = same inode; soft link = path.
- Disk (head 53, queue 98,183,37,122,14,124,65,67, moving up): FCFS 640, SSTF 236, SCAN 331, LOOK 299.
- RAID 0 stripe (no safety), 1 mirror, 5 single parity, 6 double parity, 10 mirror+stripe. RAID ≠ backup.
- Polling vs interrupts; **DMA** moves blocks, one interrupt at the end.
- Blocking / non-blocking (`EAGAIN`) / multiplexing / async (io_uring). **select O(n), 1024 limit; poll O(n); epoll O(ready)**.

**Backend extras**
- GC: ref counting (cycles problem), mark-sweep, generational, concurrent tri-color (Go). GC pauses → **p99**.
- VM = own kernel on hypervisor; container = process on shared kernel. **Namespaces = walls, cgroups = budgets.** OOM-kill, CPU throttling → p99.

**Your project lines (say at least one per answer where it fits)**
- **Actor model** per match + channels → no locks on game state; CI runs **27 Go test suites with `-race`**.
- **PostgreSQL deadlock** (200 buyers, 20 items, 0 succeeded) → fixed lock order (breaks circular wait) + conditional update (**129 ms** vs 832–984 ms) + **jittered retry** (recovery).
- Cutting off lagging clients → **livelock-like** → **throttle instead of disconnect**.
- **Linux socket buffers** absorbed hundreds of snapshots → clients seconds behind while app queue looked healthy.
- TTL chat: **2 goroutines per WebSocket** (reader + single writer); goroutines ~2 KB stack, M:N, parked on epoll.
- **vLLM PagedAttention** = OS paging for the KV cache: blocks + block table, waste 60–80% → under 4%, copy-on-write for shared prefixes.
