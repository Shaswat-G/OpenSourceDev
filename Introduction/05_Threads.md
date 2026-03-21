# Processes and Threads

## 1. From Program to Process

A **program** is a passive artifact — bytes sitting on disk. For compiled languages (C, C++, Rust) this is a binary executable. For interpreted languages (Python, JavaScript) the *interpreter itself* is the program; your `.py` file is just text that the interpreter reads into its own memory.

A **process** is what a program becomes when it runs. The OS does not simply start executing bytes from disk — it constructs a controlled execution environment around the program. That environment has four essential components:

| Component | What it contains |
|---|---|
| **Execution context** | CPU register state (instruction pointer, stack pointer, general registers, flags) |
| **Address space** | The process's private virtual memory layout |
| **Resources** | Open files, sockets, pipes, devices |
| **Identity & metadata** | PID, parent/child relationships, security credentials |

The key property of a process is **isolation**: each process sees only its own memory. Two processes cannot directly read or write each other's data. This is enforced in hardware by the MMU, which gives every process a private virtual address space.

### The Virtual Address Space

Every process sees a flat, private address range (64-bit on modern systems). From low to high addresses, it is organised as:

```
High addresses
┌─────────────────┐
│     Stack       │  Local variables, function call frames — grows downward
├─────────────────┤
│       ↕         │  (unmapped gap)
├─────────────────┤
│      Heap       │  Dynamically allocated memory (malloc/new) — grows upward
├─────────────────┤
│   Shared libs   │  libc, libpthread, etc. (mapped at runtime)
├─────────────────┤
│   Data section  │  Global and static variables (initialized + BSS)
├─────────────────┤
│   Text section  │  Executable instructions — read-only
└─────────────────┘
Low addresses
```

This is virtual memory — the addresses a process uses are not real RAM locations. The kernel maintains **page tables** (per-process data structures) that translate virtual addresses to physical RAM addresses. The CPU's memory management unit (MMU) performs this translation on every memory access, using the page tables. The critical point: because each process has its own page tables, its virtual addresses map to different physical locations than any other process's virtual addresses. **This is how memory isolation is enforced.** A virtual address `0x7fff1234` in Process A and the same address in Process B refer to completely different physical memory.

---

## 2. The Process Control Block (PCB)

The kernel tracks every running process in a data structure called the **Process Control Block** — in Linux, this is `struct task_struct`. There is one PCB per process (or per thread — more on this shortly). It stores everything needed to suspend and later resume a process:

| Field group | Contents |
|---|---|
| **Identity** | PID, executable name, UID/GID credentials |
| **CPU state** | Instruction pointer (RIP), stack pointer (RSP), general-purpose registers, CPU flags — the complete snapshot of where the CPU was when this process last ran |
| **Memory** | Pointer to the page tables for this process's address space |
| **Resources** | File descriptor table (stdin=0, stdout=1, stderr=2, and all other open files/sockets) |
| **Scheduling** | State (running/ready/blocked), priority, CPU time consumed |
| **Process tree** | Pointer to parent PCB, list of children |

### Process States

```
               fork()
                 │
                 ▼
              [NEW] ──────────────────────────────────────────────►  [ZOMBIE]
                 │                                                       ▲
           admitted                                                   exit()
                 │                                                       │
                 ▼                                                       │
     ┌──────► [READY] ◄──────────── I/O complete ───────────────── [RUNNING]
     │           │                                                       │
     │       scheduler                                              wait for I/O
     │       dispatches                                                  │
     │           │                                                       ▼
     │           └─────────────────────────────────────────────► [BLOCKED]
     │
  preempted
```

| State | Meaning |
|---|---|
| **Running** | Currently executing on a CPU core |
| **Ready** | Runnable, waiting its turn on the CPU |
| **Blocked** | Waiting for an external event (disk I/O, network, lock) — *cannot run even if the CPU is free* |
| **Zombie** | Has exited, but its PCB remains until the parent calls `wait()` to collect its exit status |
| **Stopped** | Suspended by a signal (e.g., `SIGSTOP`, debugger attach) |

---

## 3. Concurrency, Parallelism, and Context Switching

**Concurrency** is the *appearance* of simultaneous execution, achieved on a single core by rapidly switching between processes. The CPU runs process A for a time slice, saves its state, runs process B, saves its state, returns to A, and so on — fast enough that to a human it looks simultaneous.

**Parallelism** is *actual* simultaneous execution on multiple cores. If a system has N cores, up to N processes can genuinely run at the same moment.

**Context switching** is the mechanism that enables concurrency. When the scheduler decides to switch from process A to process B:

1. The CPU state of A (all registers, instruction pointer, flags) is saved into A's PCB.
2. The CPU state of B is loaded from B's PCB back into the physical CPU registers.
3. The MMU is told to use B's page tables (by updating the CR3 register on x86), switching the entire virtual address space.
4. Execution resumes wherever B was last interrupted.

The CPU resumes B exactly where it left off — it has no idea it was ever paused.

---

## 4. Why Processes Alone Are Not Enough

Consider a web server. When 100 clients connect simultaneously, the naive single-process model processes them one at a time — while waiting for a disk read for client 1, client 2 sits idle even though the CPU is free. The CPU is wasted.

The natural fix seems to be: spawn a new process per client. Each process can run independently, blocking on its own I/O without affecting others. This works, but it has serious costs:

- **Process creation is expensive.** The OS must duplicate the entire address space (all page tables), allocate new kernel stacks, copy file descriptors, and initialise a full PCB.
- **Memory overhead.** Even with copy-on-write optimisations, each process maintains its own independent memory image.
- **Communication is hard.** Processes are isolated by design. Sharing state between them requires explicit IPC (pipes, shared memory, sockets) — complex to program correctly.

The insight that resolves this is: processes are expensive because they own *everything* — address space, file descriptors, credentials, kernel state. What if we had lightweight execution units that shared most of that overhead but could still run independently?

---

## 5. Threads: Concurrent Execution Within a Process

A **thread** is an independent execution path *inside* a process. Multiple threads share the same address space, file descriptor table, and credentials, but each has its own:

- **Stack** — local variables and call frames are private per thread
- **CPU register state** — each thread has its own instruction pointer, stack pointer, and general-purpose registers
- **Scheduling state** — the kernel can independently schedule, block, and wake each thread

This sharing is the key efficiency: creating a thread does not duplicate the address space or any kernel resources. The kernel simply allocates a new stack and a new register context and inserts the thread into the scheduler. The cost is a fraction of a process creation.

```
Process
├── Address Space (shared by all threads)
│   ├── Text section        ← all threads execute code from here
│   ├── Data section        ← global variables: shared, needs synchronisation
│   ├── Heap                ← dynamically allocated: shared, needs synchronisation
│   ├── Stack of Thread 1   ← private
│   ├── Stack of Thread 2   ← private
│   └── Stack of Thread N   ← private
└── Resources (shared)
    ├── File descriptor table
    ├── Credentials (UID/GID)
    └── Signal handlers
```

Each thread has its own kernel stack as well — used when that thread makes a system call or handles a fault, since kernel code cannot safely use user-provided stacks.

### Threads and the Linux Kernel

Linux does not have a separate "thread" struct. The kernel represents every unit of execution — whether a thread or a process — as a **task** (`struct task_struct`). The distinction between a process and a thread is simply which resources are *shared* vs. *copied* at creation time, controlled by flags passed to the `clone()` system call. `pthread_create()` in userspace ultimately calls `clone()` with flags that share the address space and file descriptor table, producing what we call a thread. `fork()` calls `clone()` with flags that copy them, producing what we call a process.

---

## 6. Why Process Creation Requires a System Call

A user-space process cannot create another process by itself. This is a physical constraint, not just a policy:

Every process operates in a virtual address space. The hardware boundary between user mode (Ring 3) and kernel mode (Ring 0) exists precisely to protect kernel structures. The actions required to create a new process — allocating a new page table, writing to the hardware register that controls address translation (CR3), inserting a new task into the scheduler's run queue, allocating a PID from the global namespace — all require Ring 0 privilege.

The interface between user space and kernel space is the **system call**. A system call is a controlled, hardware-mediated request: the user program places a call number and arguments in specific CPU registers, then executes a special instruction (`syscall` on x86-64) that atomically switches to kernel mode at a pre-defined kernel entry point. The kernel validates and handles the request, then returns to user mode. The user program has no way to jump to arbitrary kernel code — it can only enter through this controlled gate.

Relevant process/thread system calls:

| Call | What it does |
|---|---|
| `fork()` | Create a child process (copy of parent) |
| `execve(path, argv, envp)` | Replace the current process image with a new program |
| `clone(flags, ...)` | Create a new task with fine-grained control over what to share |
| `exit(status)` | Terminate the current process |
| `wait()/waitpid()` | Block until a child terminates; collect its exit status |
| `getpid()` / `gettid()` | Return the process ID / thread ID |
| `kill(pid, signal)` | Send a signal to a process |

---

## 7. The Fork–Exec Model

Unix process creation uses a two-step pattern that separates *cloning* from *loading a new program*.

### `fork()` — Clone the Parent

`fork()` creates a child process that is an almost-complete copy of the parent. After `fork()` returns, there are two processes with identical memory contents, file descriptors, and CPU state. Both processes resume execution at the instruction immediately after the `fork()` call.

The only difference visible to program code: `fork()` returns the child's PID to the parent, and returns **0** to the child. This is the standard idiom:

```c
pid_t pid = fork();

if (pid == 0) {
    // We are the child — pid is 0
    execve("/bin/ls", args, env);  // replace ourselves with ls
} else {
    // We are the parent — pid holds the child's PID
    waitpid(pid, &status, 0);      // wait for child to finish
}
```

**Copy-on-Write (CoW):** Duplicating the entire address space on every `fork()` would be prohibitively expensive. Instead, the kernel marks all pages as shared and read-only in both parent and child. Physical memory is only copied when either process *writes* to a page, triggering a page fault that the kernel handles by making a private copy for the writing process. In the common `fork()` → `execve()` pattern, the child replaces its entire address space immediately, so most pages are never actually copied.

### `execve()` — Replace the Process Image

`execve(path, argv, envp)` does not create a new process. It *replaces* the calling process with a new program. The kernel:

1. Opens and validates the specified ELF binary.
2. Discards the current address space.
3. Maps the new program's code, data, and stack into memory.
4. Sets the instruction pointer to the new program's entry point (`_start`).
5. Execution continues in the new program — the previous code is gone.

Since `execve()` resets the instruction pointer, it never returns to the caller (unless it fails with an error).

### `posix_spawn()` — Combined Convenience

`posix_spawn()` is a POSIX interface that performs `fork()` + `execve()` atomically from the caller's perspective. It is safer in multi-threaded programs (raw `fork()` in a multi-threaded process has subtle hazards — only the calling thread survives in the child, but mutexes held by other threads remain locked forever in the child).

---

## 8. Thread Creation and the Unified `clone()` Interface

In Linux, creating a thread and creating a process are both done via `clone()` — the difference is purely in the flags:

| Creation type | Key flags passed to `clone()` | Effect |
|---|---|---|
| **Process** (`fork`) | `SIGCHLD` only | New address space, new FD table, new signal handlers |
| **Thread** (`pthread_create`) | `CLONE_VM \| CLONE_FILES \| CLONE_SIGHAND \| CLONE_THREAD` | Share address space, FD table, signal handlers; same thread group |
| **Container process** | `CLONE_NEWPID \| CLONE_NEWNET \| ...` | New PID namespace, new network namespace (isolation primitives) |

`pthread_create()` in userspace (glibc) calls `clone()` with the thread flags, allocates a new stack in the process's address space, and passes it to the kernel. The kernel creates a new `task_struct`, initialises a new CPU context starting at the thread function, and adds it to the scheduler.

---

## 9. Multi-Core Systems and Parallelism

A **CPU core** is one independent processing unit. Modern systems have:
- Multiple cores per CPU die (each with private L1/L2 caches, shared L3)
- Potentially multiple CPU sockets on one motherboard
- The OS sees each core as a separate processor

**Parallelism** — true simultaneous execution — is only possible with multiple cores. If a system has N cores, at most N threads run simultaneously. Threads beyond N compete for time slices.

**Concurrency vs. Parallelism, precisely:**

| Concept | Definition | Requires |
|---|---|---|
| Concurrency | Multiple tasks making progress, possibly interleaved | 1 or more cores |
| Parallelism | Multiple tasks executing at the same physical instant | 2+ cores |

Concurrency is a *program structure* property; parallelism is a *hardware execution* property. A concurrent program may or may not run in parallel depending on available cores.

**Thread count strategy:** A common guideline is to use as many threads as logical cores for CPU-bound work (more threads just add context-switch overhead). For I/O-bound work (where threads spend time blocked), more threads than cores is beneficial — a blocked thread consumes no CPU while another runs.

**Two flavours of parallelism:**

- **Data parallelism:** Partition a dataset and apply the same operation on each partition across cores. Example: parallel matrix multiplication, SIMD operations.
- **Task parallelism:** Distribute different operations (possibly heterogeneous) across cores. Example: one thread handles networking while another compresses data.

---

## 10. Complete Mental Model

```
Operating System
│
├── Scheduler
│   └── Run Queue: [Task A] [Task B] [Task C] ...
│       (each task is a task_struct / PCB)
│
├── Process P
│   ├── PID, credentials, signal handlers
│   ├── Address Space (mm_struct)
│   │   ├── pgd ──► page tables ──► physical RAM
│   │   ├── Text, Data, Heap regions
│   │   └── Per-thread stacks
│   ├── File Descriptor Table [0:stdin, 1:stdout, 2:stderr, ...]
│   │
│   ├── Thread T1 (main thread)
│   │   ├── task_struct (pid == tgid for main thread)
│   │   ├── CPU state: RIP, RSP, registers (saved when not running)
│   │   ├── User stack (in process address space)
│   │   └── Kernel stack (used during system calls)
│   │
│   └── Thread T2 (created by T1 via clone/pthread_create)
│       ├── task_struct (pid = TID, tgid = process PID)
│       ├── CPU state: own RIP, RSP, registers
│       ├── Own user stack
│       └── Own kernel stack
│
└── Process Q (isolated — different mm_struct, different page tables)
    └── ...
```

**Key invariants to internalise:**

1. **Processes are the isolation boundary.** Different processes cannot touch each other's memory without explicit kernel mediation (IPC).
2. **Threads are the execution boundary.** Within a process, threads share memory and can touch each other's heap and globals freely — but this requires synchronisation.
3. **The scheduler operates on tasks.** In Linux, both processes and threads are scheduled as tasks. A process with 4 threads contributes 4 tasks to the run queue.
4. **A context switch on a thread within the same process is cheaper** than a full process context switch, because the address space (page tables, TLB entries) does not need to change.
5. **Every task has two stacks** — one user stack (in the process's virtual address space) and one kernel stack (in kernel memory), because kernel code cannot trust user-provided memory for its own execution.



## Inter Process Communication
Independent Process can run independently while cooperating processes need to share data with each other.
Cooperation can lead to faster computation (data parallelism) or divison of labor and specialization (modularity with task parallelism)
Thus, we need a mechanism for processes to **coordinate** and **synchronize**.

There are boradly only 2 solutions:

### 1. Shared Memory
Usually, the OS together with the MMU and page tables virtualize the memory that the process sees and enforeces memory isolation (one process can not r/w/x any other memory address outside its own address space layout). However, in the memory sharing model, using system calls, one process can agree to share a memory region with another so both processes can interact (r/w/x) with the shared region.

However, what is written, at what address and how it is read is not managed by the OS, but instead, by the processes.
This can cause issues (Eg: in a producer consumer arch, a producer process writes 8 bit signed integers to a buffer in this shared space, while a consumer
process reads it as 16 bit unsigned integers, completely misreading the data.). This can cuase undefined behavior.

Eg: The Chromium browser runs as (1 web browser process responsible for UI and user I/O, n renderer processes for n tabs, and m plugin processes for m plugins)

This is fast for sharing large amounts of data.

### 2. Message Passing
Writing and reading from shared memory causes problems (RACE conditions, will discuss in thread sync) among others, so the OS provides a message passing
mechanism as an alternative. Primitives are:
1. Pipes: Unidirectional communication link (using pipe() sys call in C), There is something called FIFO pipes which are named files that allow bidirectional communcaiton.
2. Sockets: Client server architecture.
3. RPCs: A Protocol that helps one process on machine to request service from a process in another machine without understanding network details. The message has to well structured, it has tobe addressed to the RPC listener daemon on the remote server, with the identifier of the function / resource and parameters for the same. The function is then executed on the remote server and sent back to the client. The detials of the network are hidden and abstracted via a remote-proceedure "stubs" on the client side. The stub  locates the port on the server and amrshals (packaging params in a way that can be sent over a network, it handles serialization and deserialization) the parameters. A server side stub will receievd, unpack and pass this info to the server process. The core idea is to hide and abstract away the compleixty of talking over a network and expose this to something which looks like a local function call.

All of these follow the same basic structure. A system call to open such a connection (or logical link) is requested by one process, and this link, once established forms the channel to send messages. At the physical level, the kernel creates a queue of messasges in its own address space (priveleged), and therefore the kernel has to expose system calls like send(to_process, message) and receive(from_process). The behavior of the queque can change with requirements -> sync vs async, bucket or buffer style (unbounded vs bounded) etc. For full duplex communication, we can have 2 message queues in the kernel space so each process can send and receive messages.

1. Naming: Under direct communication, process P and Q; P can send a message like send(Q, message), and Q can receive a message form P, receive(P). With indirect communication, we can think of posting messages on a mailbox and reading messages from the mail box.
2. Blocking / Non-blocking: this is async vs sync communication, do one, both or none of sender and receiver wait (block) for messages dispatch and receipt. The queue size is 0 (for perfect sync), bounded or unbounded.

Such mailboxes are called ports. 

A process could have a general port (listening) so any other process can send a conneciton request to this process and then open a separate secure port for communcations.

The logical architecture of message passing can thus be extended beyond a single core CPU + RAM computer to different computers on a network, with the added complexity of device drivers, NIC programming.

The client server architecutre is built on "sockets" (HTTP, HTTPS, FTP, SSH). In such cases, the IP address identifies the machine, and port identifies the mailbox to send the message to.

TCP vs UDP?

Transport with HTTP1.0/2.0/3.0 ? What are these