# Process Creation and Management

## 1. Concepts and Terminology

1. A **program** is a **passive executable artifact stored on disk** consisting of machine code and static data. For compiled languages its the compiled executable binary that is assembled and executed. For interpreted languages (python, javascript), the interpreter is the program which runs the main function. A program becomes a **process** when executed.

2. A **process** is a **kernel-managed resource container representing an executing program instance**, including execution state, memory environment, and owned resources.

| Component         | Description              |
| ----------------- | ------------------------ |
| Execution context | CPU register state, flags       |
| Address space     | virtual memory layout    |
| Resources         | files, sockets, network connections, devices  |
| Metadata          | PID, parent, child credentials |


3. A **thread** is an **independent execution path within a process**. Threads share address space, file descriptors, credentials but have their own: stack, CPU registers and scheduling state.

4. A **PCB** is a kernel data structure that stores all information about a process, keyed by PID.

| Field                 | Meaning                 |
| --------------------- | ----------------------- |
| PID                   | process identifier      |
| state                 | running, ready, blocked |
| register state        | CPU context             |
| memory descriptor     | page tables             |
| file descriptor table | open resources          |
| credentials           | UID/GID                 |

Typical states:

| State   | Meaning               |
| ------- | --------------------- |
| Running | executing on CPU      |
| Ready   | waiting for CPU       |
| Blocked | waiting for I/O       |
| Stopped | suspended             |
| Zombie  | exited but not reaped |


5. A **PID** is a **namespace-unique integer identifier** referencing a process. Allows sending signals like SIGTERM and SIGKILL, debugging, monitoring and process control

```
kill 1234
ps -p 1234
waitpid(1234)
```

6. Each process sees a **private memory space**, with a typical layout as follows. Memory isolation is enforced via **page tables + MMU**.

```
High addresses
┌──────────────┐
│ Stack        │
├──────────────┤
│ Heap         │
├──────────────┤
│ Shared libs  │
├──────────────┤
│ Data section │
├──────────────┤
│ Code section │
└──────────────┘
Low addresses
```
7. Page tables are Kernel-managed data structures that map Virtual Address → Physical Address. This virtualization creates memory isolation for memory security.

8. The MMU is a CPU hardware responsible for translating virtual addresses into physical addresses. It contains a CR3 register that stores a pointer to the page tables for each process.


9. Each thread has two stacks, since user memory cannot be trusted by kernel code.

| Stack        | Used for                  |
| ------------ | ------------------------- |
| User stack   | program execution         |
| Kernel stack | system calls / interrupts |

10. Processes interact with OS resources through **file descriptors**.
```
0 → stdin
1 → stdout
2 → stderr
```
They can also represent files, sockets, pipes and devices.

11. The **scheduler** determines which runnable process gets CPU time (by scheduling policies) by maintaining a **run queue** with PCBs of each process. 
Modern schedulers include fairness algorithms (e.g., Linux CFS).

Data structures:

```
Run Queue
Ready Tasks
Scheduling policies
```

## Process Creation Workflow

When launching a program the OS must establish a reliable, unique, referenceable identity (PID), execution context (CPU registers + flags), memory environment (CR3 in MMU and pagetables), resource ownership (UID, GID, cgroups) and scheduling eligibility.

1. Allocate PID
2. Create Process Control Block
3. Process enters scheduler structures.
4. Create Address Space layout (page tables)
5. POint CR3 to page tables (MMU)
6. Allocate Kernel Stack for each thread
7. Initialize CPU Register Context
8. Assign Security Credentials (inherited from parents)
9. Attach Resource Control Structures (cgroups, namespaces)
10. Configure Signal / Exception Handling by init kernel strcutures for signals, faults, async events
11. Initialize File Descriptor Tables (integers mapped to stdin, stdout, resources, sockets, devices)
12. Register Resource Accounting
13. Establish Parent–Child Relationship
14. Insert into System Process Tables to make process visible to system utilities.
15. Context Switch to Process (save old register, load new register, switch page tables)
16. Expose Through Kernel Interfaces

## Concurrency and Parallelism

**Concurrency** : Multiple processes *appear* to run simultaneously via *time slicing*.
**Parallelism** : Processes truly run simultaneously on multiple cores.
**Context Switching**: Changes CPU register state,  memory environment (page tables), scheduler ownership


## Fork–Exec Model (Unix)

1. *fork()* Creates a **child process** by duplicating the parent (also duplicates the execution context). Instead of copying memory immediately, the parent pages are shared and marked read-only and are Copied only when modified.

2. *execve()* Replaces process memory with a new program.



## Mental Model

A **process** can be understood as:

```
Process
 ├── Execution Context (registers, stacks)
 ├── Memory Environment (address space)
 ├── Resource Container (files, sockets)
 ├── Security Identity (UID/GID)
 └── Kernel Metadata (PID, scheduling)
```


Explain all the technical terms with their level of abstraction - logical, physical.

Eg: fork, spawn, process

What is multi-process architecture?

An application is composed of multiple processes running in the background. Their interaction (communication) gives rise to the (emergent) phenomenon we see (application). A user only interact with the application via GUI / cmd.

Applications, once launched can programmatically spawn multiple processes at any point during their execution.


How does a parent process p1 create a child process p2?

To start a new process, the new process's executable code (binary) needs to be written to the address space so that the cpu can start executing it step by step. The complication is that one process is isolated from other processes via (memory virtualization of address space -- verify this and add more detail to explain this clearly with examples), and therefore any other memory is completely inaccessible to this p1?

And so although p1 can read p2's executable code in its own address space, it cannot place this in any other address space outside of it.

More concretely, here is why a process in user space cannot create another process in the user space:

1. Allocate a PID form the global PID namespace.
2. Create a kernel process object (Process control block)
3. Insert the process into kernel scheduler run queues 
4. Allocate and initialize page tables for a new address space
5. Program the MMu with those page tables
6. Allocate kernel stacks for the new execution context
7. Initialize CPU context (register frame, privelege level, flags)
8. Assign security credentials (UID/GID, token, capabilities)
9. Attach the process to namespaces / job objects / cgroups
10. Set up signal / exception / APC delivery structures
11. Duplicate or initialize file descriptor tables
12. Register the process with resource accounting and limits
13. Link parent / child relationships in the process tree
14. Make the process visible to kernel and other processes
15. Allow the CPU to context switch into the new process
16. Expose the process through kernel APIs (/proc, task lists, handles)

# Linux Process Management: A Kernel-Grounded Reference

> All function signatures, struct definitions, and source references are anchored to the mainline Linux kernel (`kernel/fork.c`, `include/linux/sched.h`, `arch/x86/kernel/process_64.c`, `arch/x86/entry/entry_64.S`). Where behavior differs between kernel versions this is noted.

---

## Part 1 — Core Abstractions and Their Levels

### 1.1 Program vs. Process vs. Thread — Logical and Physical

**Program (logical):** A named, static artifact on the filesystem — a sequence of instructions and data that describes *what to compute*. For compiled languages (C, C++, Rust) this is an ELF binary. For interpreted languages (Python, JavaScript) the *interpreter binary* is the program; the `.py` source file is just text loaded into the interpreter's heap at runtime.

**Program (physical):** Bytes residing on a block device, organized by a filesystem. Nothing executes until the kernel maps it into memory.

**Process (logical):** A running instance of a program — an isolated, kernel-managed execution environment. The fundamental resource-ownership unit. Isolation is the key property: two processes cannot directly read each other's memory.

**Process (physical):** A `struct task_struct` in kernel memory, associated page tables in RAM, a kernel stack per thread, an entry in the CPU scheduler's run queue, and zero or more hardware TLB entries.

**Thread (logical):** An independent execution path *within* a process. Threads share the same address space, file descriptor table, and credentials, but each has its own stack, CPU register state, and scheduling quantum.

**Thread (physical — Linux-specific):** In Linux there is **no separate `thread` struct**. The kernel's basic scheduling unit is called a *task*, represented by `struct task_struct`. Both processes and threads are tasks. Two tasks that belong to the same process simply share the same `mm_struct` (address space), `files_struct` (FD table), and `signal_struct`. This is a critical design decision — it makes `clone()` the unified primitive.

---

## Part 2 — `struct task_struct`: The Kernel's PCB

**Source:** `include/linux/sched.h`

The `task_struct` is the most important data structure in the Linux kernel. It is the PCB. Every thread and process is a `task_struct`. As of kernel 6.x it is over 9 KB and referenced in nearly 1,400 source files. Below is a logically grouped, annotated excerpt of its most important fields.

### 2.1 Identity and State

```c
struct task_struct {
    /*
     * ----- IDENTITY -----
     * pid: confusingly named. In Linux, task_struct represents a *thread*.
     * For the main thread of a process, pid == tgid.
     * For additional threads, pid is the thread ID (TID), tgid is the process ID.
     * tgid (Task Group ID) is what userspace calls getpid().
     */
    pid_t                   pid;    /* Thread ID (TID at kernel level) */
    pid_t                   tgid;   /* Thread Group ID = true PID seen by userspace */
    char                    comm[TASK_COMM_LEN]; /* Executable name, max 16 chars */

    /*
     * ----- STATE -----
     * Represented as a bitmask. The volatile qualifier is required because
     * an interrupt handler can modify this field asynchronously.
     */
    volatile long           state;
    /*
     * State values (defined in include/linux/sched.h):
     *
     * TASK_RUNNING        (0)  — runnable: either on CPU or on run queue
     * TASK_INTERRUPTIBLE  (1)  — sleeping; can be woken by a signal or event
     * TASK_UNINTERRUPTIBLE(2)  — sleeping; cannot be woken by a signal (e.g., disk I/O wait)
     * TASK_ZOMBIE         (4)  — has called exit(), waiting for parent to wait()
     * TASK_STOPPED        (8)  — stopped by SIGSTOP or a debugger (ptrace)
     *
     * Important: TASK_RUNNING does NOT mean "on the CPU right now."
     * It means "eligible to run." A task is truly running only when the
     * scheduler picks it from the run queue.
     */
```

### 2.2 Scheduling Fields

```c
    /*
     * ----- SCHEDULING -----
     * The scheduler sees tasks through these fields.
     * Linux CFS (Completely Fair Scheduler) uses virtual runtime (vruntime)
     * to select the next task: the task with the smallest vruntime runs next.
     */
    int                     prio;          /* Effective dynamic priority (100–139 for normal tasks) */
    int                     static_prio;   /* Computed from nice value: 100 + (nice + 20) */
    int                     normal_prio;   /* Priority without any real-time boost */
    unsigned int            rt_priority;   /* Real-time priority (1–99 for SCHED_FIFO/SCHED_RR) */
    unsigned int            policy;        /* SCHED_NORMAL, SCHED_FIFO, SCHED_RR, SCHED_BATCH, etc. */
    cpumask_t               cpus_allowed;  /* Which CPU cores this task is allowed to run on */
    struct sched_entity     se;            /* CFS scheduling entity: contains vruntime */
    struct sched_rt_entity  rt;            /* Real-time scheduling entity */
```

### 2.3 Architecture-Specific CPU State: `thread_struct` and `pt_regs`

This is the section your notes are least detailed on, and it is the most mechanically important. The kernel separates CPU state into **two distinct structures** for a fundamental reason.

```c
    /*
     * ----- HARDWARE CPU STATE -----
     * Linux splits register state into two structures:
     *
     *   pt_regs      — saved when entering kernel mode (syscall / interrupt)
     *                  Lives at the TOP of the kernel stack.
     *                  Contains the FULL register snapshot of the user-mode process.
     *
     *   thread_struct — saved during a context switch (task A → task B)
     *                   Lives inside task_struct.
     *                   Contains ONLY the callee-saved registers (those that
     *                   the C ABI requires the called function to preserve).
     */
    struct thread_struct    thread; /* arch-specific; see arch/x86/include/asm/processor.h */
```

**`struct pt_regs` (x86-64) — the full user-mode register snapshot:**

Defined in `arch/x86/include/asm/ptrace.h`. Saved on the kernel stack by the `ENTRY(syscall_entry)` assembly stub (`arch/x86/entry/entry_64.S`) the instant a `syscall` instruction fires.

```c
struct pt_regs {
    /* General-purpose registers (saved in SAVE_ALL macro order) */
    unsigned long r15;
    unsigned long r14;
    unsigned long r13;
    unsigned long r12;
    unsigned long rbp;      /* Frame pointer */
    unsigned long rbx;

    /* Arguments passed to system calls per x86-64 SysV ABI:
     * rdi=arg1, rsi=arg2, rdx=arg3, r10=arg4, r8=arg5, r9=arg6
     * rax holds the syscall number on entry, return value on exit */
    unsigned long r11;
    unsigned long r10;
    unsigned long r9;
    unsigned long r8;
    unsigned long rax;      /* Syscall number / return value */
    unsigned long rcx;      /* Return address (from `syscall` instruction) */
    unsigned long rdx;
    unsigned long rsi;
    unsigned long rdi;

    /* Saved by the CPU hardware automatically on privilege-level change */
    unsigned long orig_rax; /* Original syscall number (for restart logic) */
    unsigned long rip;      /* Instruction pointer — where user code was executing */
    unsigned long cs;       /* Code segment selector (user mode = 0x33) */
    unsigned long eflags;   /* CPU flags register (carry, zero, sign, overflow, etc.) */
    unsigned long rsp;      /* User-mode stack pointer */
    unsigned long ss;       /* Stack segment selector */
};
```

**Why is `rip` the most important field here?** Because it records *exactly which user-mode instruction* was executing when the process entered the kernel. When the kernel eventually returns to user mode (`sysretq` or `iretq`), `rip` is restored and execution resumes from precisely that point.

**`struct thread_struct` (x86-64) — the context-switch register snapshot:**

Defined in `arch/x86/include/asm/processor.h`.

```c
struct thread_struct {
    /*
     * sp: Kernel stack pointer. This is the CRITICAL field for context switching.
     *
     * When the scheduler switches away from task P:
     *   __switch_to_asm saves RSP into P->thread.sp
     *
     * When the scheduler switches to task Q:
     *   __switch_to_asm loads Q->thread.sp into RSP
     *
     * This single register swap is what "switches" execution to a new task.
     */
    unsigned long           sp;         /* Saved kernel RSP */
    unsigned long           sp0;        /* Top of kernel stack (for TSS) */

    /* Segment registers (legacy x86 segmentation, still maintained for ABI) */
    unsigned short          es, ds, fsindex, gsindex;

    /* fs_base / gs_base: used for thread-local storage (TLS).
     * The FS register's base address holds the TLS block pointer.
     * glibc uses FS for pthread-local variables. */
    unsigned long           fsbase;
    unsigned long           gsbase;

    /* FPU / SSE / AVX state (lazy-saved only when another task uses FPU) */
    struct fpu              fpu;
};
```

**Why are `pt_regs` and `thread_struct` separate?** Because they are saved at *different events* for *different purposes*:
- `pt_regs` is the snapshot taken on *every kernel entry* (syscall, interrupt, fault). It preserves the complete user register state so the kernel can return exactly where it left off.
- `thread_struct` is saved only during a *context switch*. The C ABI means the caller-saved registers (RAX, RCX, RDX, RSI, RDI, R8–R11) are already saved by the compiler-generated code; only the callee-saved registers (RBX, R12–R15, RBP, RSP) need to be explicitly saved here.

### 2.4 Memory: `mm_struct`

```c
    /*
     * ----- MEMORY / ADDRESS SPACE -----
     * mm: pointer to the process's memory descriptor.
     * active_mm: for kernel threads, which have no user address space,
     *            this points to the mm of the last user task that ran.
     *            This is a performance optimization (avoids TLB flushes
     *            when switching to/from kernel threads).
     */
    struct mm_struct        *mm;
    struct mm_struct        *active_mm;
```

`struct mm_struct` (defined in `include/linux/mm_types.h`) contains:

```c
struct mm_struct {
    pgd_t                   *pgd;       /* Physical address of top-level page table.
                                         * This is what gets loaded into CR3 on a context switch.
                                         * CR3 is the x86 register that points the MMU to the
                                         * page tables for the currently running process. */
    unsigned long           mmap_base;  /* Base address for mmap'd regions */
    unsigned long           start_code, end_code;   /* Text segment bounds */
    unsigned long           start_data, end_data;   /* Data segment bounds */
    unsigned long           start_brk,  brk;        /* Heap: start and current top */
    unsigned long           start_stack;            /* Stack start address */
    atomic_t                mm_users;   /* Reference count: number of threads sharing this mm */
    atomic_t                mm_count;   /* Structural reference count */
    /* ... */
};
```

**The CR3 register is the physical link between `mm_struct` and the MMU.** On every context switch, `switch_mm()` writes `mm->pgd` into CR3. The MMU hardware then uses this page directory to translate every subsequent virtual address. This is what enforces memory isolation: two processes have different `mm_struct` instances with different `pgd` pointers, so their virtual-to-physical address maps are completely disjoint.

### 2.5 Process Tree Relationships

```c
    /*
     * ----- PROCESS TREE -----
     * Linux maintains a full tree of tasks.
     * real_parent: the biological creator (who called fork())
     * parent:      the current parent for wait() — can differ if reparented (e.g., via ptrace)
     */
    struct task_struct      *real_parent;
    struct task_struct      *parent;
    struct list_head        children;   /* Sentinel head of the list of child tasks */
    struct list_head        sibling;    /* Links this task into its parent's children list */
    struct task_struct      *group_leader; /* Points to the main thread of this process */

    /* tasks: doubly-linked list threading ALL tasks in the system.
     * Used by for_each_process() macro to iterate all processes. */
    struct list_head        tasks;
```

### 2.6 File Descriptors, Signals, Credentials

```c
    /* ----- RESOURCES ----- */
    struct files_struct     *files;     /* File descriptor table.
                                         * files->fd_array[0] = stdin
                                         * files->fd_array[1] = stdout
                                         * files->fd_array[2] = stderr
                                         * Shared across threads (CLONE_FILES) or copied. */

    struct signal_struct    *signal;    /* Signal state shared by all threads in a process */
    struct sighand_struct   *sighand;   /* Signal handler function pointers */
    sigset_t                blocked;    /* Bitmask of blocked signals */
    sigset_t                pending;    /* Bitmask of pending signals */

    /* ----- CREDENTIALS ----- */
    const struct cred       *real_cred; /* UID/GID of the task's creator */
    const struct cred       *cred;      /* Effective credentials (may differ after setuid) */
    /*
     * struct cred contains:
     *   uid_t uid, euid, suid, fsuid;   (real, effective, saved, filesystem UID)
     *   gid_t gid, egid, sgid, fsgid;
     *   kernel_cap_t cap_inheritable, cap_permitted, cap_effective; (POSIX capabilities)
     */

    /* ----- NAMESPACES / CGROUPS ----- */
    struct nsproxy           *nsproxy;  /* Pointer to namespace group (PID, net, mount, IPC, UTS, time) */
    struct css_set           *cgroups;  /* Control group membership (resource limits: CPU, memory, I/O) */
```

---

## Part 3 — Why Process Creation Requires a System Call

A user-space process *cannot* create another process on its own. This is not a policy restriction — it is a physical impossibility given memory virtualization. Here is why, precisely:

Every process operates in a **virtual address space** — a private, contiguous-looking 64-bit address range. The MMU translates every virtual address the CPU generates into a physical RAM address using the page tables pointed to by CR3. Because CR3 is a privileged register (modifiable only in Ring 0 / kernel mode), a user process literally cannot change what page tables the MMU uses — which means it cannot create a new isolated address space. The actions required for process creation (allocating a PID from the global namespace, creating a `task_struct`, building new page tables, inserting into the scheduler run queue, loading the CR3 with the new page tables, writing to the TSS) all require Ring 0 privilege. Therefore process creation *must* go through a system call.

**The system call boundary (x86-64):**

1. User code places the syscall number in `RAX` and arguments in `RDI`, `RSI`, `RDX`, `R10`, `R8`, `R9`.
2. The `syscall` instruction fires. The CPU atomically saves `RIP` (return address) in `RCX`, saves `RFLAGS` in `R11`, switches to Ring 0, and jumps to the address in `IA32_LSTAR` MSR (the kernel's syscall entry point).
3. The kernel entry stub (`arch/x86/entry/entry_64.S`) executes `swapgs` to switch the GS register to the kernel's per-CPU data (where `current_task` lives), then saves all registers as `struct pt_regs` at the top of the kernel stack.
4. The syscall handler executes in kernel mode.
5. On return, `sysretq` restores `RIP` from `RCX` and `RFLAGS` from `R11`, switches back to Ring 3, and user execution resumes.

---

## Part 4 — The `fork()` / `clone()` / `execve()` Call Chain

### 4.1 Unified Kernel Entry: `kernel_clone()` (formerly `_do_fork()`)

All process/thread creation APIs converge on a single kernel function. The call chain is:

```
User space:                        Kernel space:
fork()    ─────────────────────►  sys_fork()
vfork()   ─────────────────────►  sys_vfork()    ──► kernel_clone()  ──► copy_process()
clone()   ─────────────────────►  sys_clone()                         ──► wake_up_new_task()
pthread_create() ──► clone()   ──► sys_clone()
```

In recent kernels (5.10+) `_do_fork()` was renamed `kernel_clone()`. The interface:

```c
/*
 * kernel/fork.c
 *
 * kernel_clone_args encodes everything the caller wants:
 * - flags:       CLONE_* bitmask (what to share vs. copy)
 * - exit_signal: signal sent to parent on child exit (SIGCHLD for fork)
 * - stack:       new stack address for the child (clone/threads)
 * - parent_tid:  user pointer to write parent's view of child TID
 * - child_tid:   user pointer to write child's own TID
 * - tls:         thread-local storage descriptor (for pthreads)
 */
struct kernel_clone_args {
    u64             flags;
    int __user      *pidfd;
    int __user      *child_tid;
    int __user      *parent_tid;
    int             exit_signal;
    unsigned long   stack;
    unsigned long   stack_size;
    unsigned long   tls;
    /* ... */
};

pid_t kernel_clone(struct kernel_clone_args *args);
```

The `CLONE_*` flags are what unify processes and threads:

| Flag | Effect |
|---|---|
| `CLONE_VM` | Share `mm_struct` (address space) → makes a thread, not a process |
| `CLONE_FILES` | Share `files_struct` (FD table) |
| `CLONE_SIGHAND` | Share `sighand_struct` (signal handlers) |
| `CLONE_THREAD` | Place child in same thread group (`tgid` = parent's `tgid`) |
| `CLONE_NEWPID` | Create new PID namespace (foundation of containers) |
| `CLONE_NEWNET` | Create new network namespace |
| `SIGCHLD` (only flag) | `fork()` behavior — full process copy |

### 4.2 `copy_process()` — The Core of Process Creation

This is where all the real work happens. Source: `kernel/fork.c`.

```c
/*
 * copy_process() — creates a new task_struct as a copy of the current one
 *
 * It does NOT start the new process. It returns a pointer to the new task_struct.
 * The caller (kernel_clone) is responsible for calling wake_up_new_task() to
 * place the child on the scheduler run queue.
 */
static __latent_entropy struct task_struct *copy_process(
    struct pid *pid,
    int trace,
    int node,
    struct kernel_clone_args *args)
```

**Step-by-step execution of `copy_process()`:**

**Step 1 — `dup_task_struct(current, node)`:** Allocates a new `task_struct` and a new kernel stack. The new task_struct is an exact byte-for-byte copy of the parent's at this point, *except* the stack pointer. The kernel stack is freshly allocated (not copied). This is important: the child starts with an empty kernel stack regardless of how deep the parent's kernel stack currently is.

```c
p = dup_task_struct(current, node);
/*
 * After this:
 *   p->mm           still points to current->mm (shared, not yet copied)
 *   p->stack        points to a NEW, EMPTY kernel stack page(s)
 *   p->pid          still equals current->pid (will be replaced)
 */
```

**Step 2 — Sanity and limit checks:** Verifies that `RLIMIT_NPROC` (max processes per UID) and the system-wide `threads-max` have not been exceeded. If so, fail with `EAGAIN`.

**Step 3 — Scheduler initialization:**

```c
retval = sched_fork(clone_flags, p);
/*
 * Sets p->state = TASK_NEW (not yet on run queue — prevents premature scheduling)
 * Initializes p->se (CFS scheduling entity): sets vruntime, load weight
 * Copies scheduling policy and priority from parent
 * Does NOT place p on the run queue yet
 */
```

**Step 4 — Copy or share subsystems (controlled by `CLONE_*` flags):**

```c
retval = copy_files(clone_flags, p);     /* Share or dup files_struct */
retval = copy_fs(clone_flags, p);        /* Copy filesystem root/cwd info */
retval = copy_sighand(clone_flags, p);   /* Share or dup signal handler table */
retval = copy_signal(clone_flags, p);    /* Copy signal state */
retval = copy_mm(clone_flags, p);        /* Copy or share mm_struct (address space) */
retval = copy_namespaces(clone_flags, p);/* Copy or share namespace group */
retval = copy_io(clone_flags, p);        /* Copy I/O context */
```

The `copy_mm()` function deserves special attention:

```c
static int copy_mm(unsigned long clone_flags, struct task_struct *tsk) {
    struct mm_struct *mm, *oldmm;
    oldmm = current->mm;

    if (clone_flags & CLONE_VM) {
        /*
         * Thread creation: child shares the parent's mm_struct entirely.
         * Increment mm_users reference count.
         * The child's CR3 will point to the SAME page tables.
         * No memory is copied.
         */
        mmget(oldmm);
        mm = oldmm;
        goto good_mm;
    }

    /*
     * Process (fork) creation: duplicate the mm_struct.
     * dup_mmap() copies the Virtual Memory Areas (VMAs) list.
     * All pages are marked COPY-ON-WRITE (read-only in both parent and child).
     * Physical memory is NOT copied — only the page table entries are duplicated,
     * both pointing to the SAME physical pages, marked read-only.
     *
     * When either process writes to a page, a page fault fires.
     * The kernel's fault handler (do_wp_page) allocates a new physical page,
     * copies the content, and updates the writing process's page table entry
     * to point to the new page with write permission.
     */
    mm = dup_mm(tsk, oldmm);
}
```

**Step 5 — `copy_thread()` — the architecture-specific CPU state setup:**

This is the most subtle step. It establishes the initial CPU state of the *child* so that when the scheduler first switches to it, execution resumes correctly. Source: `arch/x86/kernel/process.c` (x86-specific).

```c
int copy_thread(struct task_struct *p, const struct kernel_clone_args *args) {
    struct pt_regs *childregs;
    struct fork_frame *fork_frame;
    struct inactive_task_frame *frame;

    /*
     * task_pt_regs(p) returns the address at the TOP of the child's kernel stack.
     * This is where pt_regs will live for this new task.
     *
     * struct fork_frame is a carefully laid out structure:
     *   ┌──────────────────────────────┐  ← top of kernel stack
     *   │  struct pt_regs   (childregs)│  ← full user-mode register snapshot
     *   ├──────────────────────────────┤
     *   │  struct inactive_task_frame  │  ← callee-saved regs for context switch
     *   │    unsigned long  r15        │
     *   │    unsigned long  r14        │
     *   │    unsigned long  r13        │
     *   │    unsigned long  r12        │
     *   │    unsigned long  bx (rbx)   │
     *   │    unsigned long  bp (rbp)   │
     *   │    unsigned long  ret_addr   │  ← return address for __switch_to_asm
     *   └──────────────────────────────┘  ← grows downward
     */
    childregs = task_pt_regs(p);
    fork_frame = container_of(childregs, struct fork_frame, regs);
    frame = &fork_frame->frame;

    /*
     * CRITICAL: ret_addr is set to ret_from_fork.
     *
     * When the scheduler first runs this child (__switch_to_asm pops the stack),
     * the CPU will "return" to ret_from_fork — NOT to wherever the parent was.
     * ret_from_fork then calls schedule_tail() and eventually returns to user mode.
     */
    frame->ret_addr = (unsigned long) ret_from_fork;
    frame->bp = encode_frame_pointer(childregs);

    /*
     * Save the kernel stack pointer into thread.sp.
     * __switch_to_asm uses this to switch stacks during context switch.
     */
    p->thread.sp = (unsigned long) fork_frame;

    if (unlikely(p->flags & PF_KTHREAD)) {
        /* Kernel thread: no user-space pt_regs needed */
        memset(childregs, 0, sizeof(struct pt_regs));
        /* fn and arg stored in r12, r13 — retrieved by ret_from_fork */
        return 0;
    }

    /*
     * User-space child: copy the parent's pt_regs.
     * This makes the child's register state identical to the parent's
     * AT THE MOMENT fork() was called.
     */
    *childregs = *current_pt_regs();

    /*
     * The return value of fork() in the child must be 0.
     * RAX is the return value register in the x86-64 SysV ABI.
     */
    childregs->ax = 0;

    /* User stack pointer, segment registers are already copied from parent */
}
```

**Step 6 — Allocate PID:**

```c
pid = alloc_pid(p->nsproxy->pid_ns_for_children, args->set_tid, args->set_tid_size);
p->pid = pid_nr(pid);       /* kernel-global PID */
/* tgid: for a new process, tgid = pid; for a thread, tgid = parent's tgid */
if (clone_flags & CLONE_THREAD)
    p->tgid = current->tgid;
else
    p->tgid = p->pid;
```

**Step 7 — Link into process tree and hash tables:**

```c
/* Set parent pointers */
p->real_parent = current;
p->parent_exec_id = current->self_exec_id;

/* Add to parent's children list (protected by tasklist_lock) */
list_add_tail(&p->sibling, &p->real_parent->children);

/* Insert into PID hash table (used for fast PID lookup: kill, waitpid) */
attach_pid(p, PIDTYPE_PID);
attach_pid(p, PIDTYPE_TGID);

/* Add to global task list (for_each_process / /proc iteration) */
list_add_tail_rcu(&p->tasks, &init_task.tasks);
```

**Step 8 — `wake_up_new_task(p)` (back in `kernel_clone()`):**

```c
/*
 * Changes p->state from TASK_NEW → TASK_RUNNING.
 * Selects the correct CPU (for SMP systems).
 * Calls activate_task() → enqueue_task() to place p on the CFS run queue.
 * Calls check_preempt_curr() — if the child has higher priority, sets
 * TIF_NEED_RESCHED on the current CPU, triggering a preemption.
 *
 * Deliberately, the kernel runs the CHILD first. Reason: in the common
 * fork-exec pattern, the child immediately calls execve(), discarding all
 * the CoW pages. If the parent ran first and modified pages, those CoW
 * copies would be created needlessly.
 */
wake_up_new_task(p);
```

### 4.3 `execve()` — Replacing the Process Image

After `fork()`, the child typically calls `execve()` to load a new program. Source: `fs/exec.c`.

```c
int do_execveat_common(int fd, struct filename *filename,
                       struct user_arg_ptr argv,
                       struct user_arg_ptr envp,
                       int flags)
```

`execve()` does not create a new process. It *replaces* the calling process:

1. **`open_exec()`:** Opens and validates the target ELF binary.
2. **`bprm_mm_init()`:** Creates a new, empty `mm_struct` for the process.
3. **`copy_strings()`:** Copies `argv` and `envp` into the new address space.
4. **`search_binary_handler()`:** Finds the appropriate binary format handler (ELF, script, etc.).
5. **`load_elf_binary()`:** Parses the ELF header, maps program segments (`PT_LOAD`) into the new address space, sets up the stack, maps the dynamic linker (`ld.so`) if needed.
6. **`start_thread()`:** Sets `childregs->ip` (the instruction pointer in `pt_regs`) to the ELF entry point, and `childregs->sp` to the top of the new stack.

```c
/* arch/x86/kernel/process_64.c */
void start_thread(struct pt_regs *regs, unsigned long new_ip, unsigned long new_sp) {
    regs->ip  = new_ip;   /* ELF entry point (e.g., _start in the binary) */
    regs->sp  = new_sp;   /* Top of newly set up user stack */
    regs->cs  = __USER_CS; /* x86-64 user code segment selector (0x33) */
    regs->ss  = __USER_DS; /* x86-64 user data/stack segment selector (0x2b) */
    /* loadsegment(fs/gs/es/ds, 0) — clear segment register bases */
}
```

The old `mm_struct` is released (decrement `mm_users`; free if zero). Since `pt_regs` is modified to point to the new program, when `execve()` returns from kernel mode via `sysretq`, the CPU resumes execution at the new ELF entry point — the program's `_start` symbol — rather than at the instruction after the `execve()` syscall.

---

## Part 5 — Context Switching: `__switch_to_asm` (x86-64)

When the scheduler selects a new task, `context_switch()` is called. The x86-64 assembly in `arch/x86/entry/entry_64.S`:

```asm
/* __switch_to_asm(struct task_struct *prev, struct task_struct *next)
 * Switches execution from `prev` to `next`.
 * Returns `prev` in RAX (used by the scheduler to finish cleanup).
 */
ENTRY(__switch_to_asm)
    /* 1. Save callee-saved registers of `prev` onto prev's kernel stack */
    pushq   %rbp
    pushq   %rbx
    pushq   %r12
    pushq   %r13
    pushq   %r14
    pushq   %r15

    /* 2. Save prev's kernel stack pointer into prev->thread.sp */
    movq    %rsp, TASK_threadsp(%rdi)   /* TASK_threadsp = offsetof(task_struct, thread.sp) */

    /* 3. Load next's kernel stack pointer — THIS IS THE ACTUAL STACK SWITCH */
    movq    TASK_threadsp(%rsi), %rsp   /* RSP now points to next's kernel stack */

    /* 4. Restore callee-saved registers of `next` from next's kernel stack */
    popq    %r15
    popq    %r14
    popq    %r13
    popq    %r12
    popq    %rbx
    popq    %rbp

    /* 5. Return to __switch_to() (C function) which handles the remaining
     * non-register state: FPU, TLS (FS base), debug registers, TSS.sp0
     * (so the CPU knows where to put the kernel stack pointer on the NEXT
     *  syscall from `next`'s user-mode code).
     */
    jmp     __switch_to
END(__switch_to_asm)
```

**What happens when the child runs for the first time?** When `__switch_to_asm` pops the registers and "returns," it follows the `ret_addr` that `copy_thread()` placed on the child's kernel stack. That address is `ret_from_fork`:

```asm
ENTRY(ret_from_fork)
    LOCK ; bts $TIF_FORK, TI_flags(%r8)  /* Mark as freshly forked */
    call    schedule_tail       /* Release the runqueue lock from wake_up_new_task */
    /* If this is a kernel thread, jump to the thread function (stored in rbx/r12) */
    testq   %rbx, %rbx
    jnz     1f
    /* User process: return to user mode by restoring pt_regs */
    /* pt_regs->ax = 0 (set in copy_thread), so fork() returns 0 in child */
    jmp     ret_to_user
1:  /* Kernel thread */
    call    *%rbx               /* Call the kernel thread function */
    ...
END(ret_from_fork)
```

---

## Part 6 — The Full Process Creation Sequence (Annotated)

This is the corrected and complete version of the 16-step list in your notes, now grounded in the actual kernel functions:

| Step | Kernel Function | What Actually Happens |
|------|----------------|----------------------|
| 1 | `copy_process()` / sanity checks | Verify `RLIMIT_NPROC`, `threads-max`, signal state |
| 2 | `dup_task_struct()` | `kmem_cache_alloc` a new `task_struct`; allocate a fresh kernel stack (`alloc_thread_stack_node`); copy parent's `task_struct` fields |
| 3 | `sched_fork()` | Initialize `se` (CFS entity); set `p->state = TASK_NEW`; copy scheduling policy; do NOT enqueue yet |
| 4 | `copy_mm()` | For fork: `dup_mmap()` — copy VMA list, mark all private pages CoW; set `p->mm->pgd` to new page directory. For clone(`CLONE_VM`): `mmget()` — share `mm_struct`, same `pgd` |
| 5 | `mm_struct->pgd` loaded into MMU | Not at creation time — happens at first context switch via `switch_mm()` → `write_cr3(mm->pgd)` |
| 6 | `copy_thread()` | Allocate child's `fork_frame` atop kernel stack; copy parent's `pt_regs`; set `childregs->ax = 0`; set `frame->ret_addr = ret_from_fork`; save stack pointer into `p->thread.sp` |
| 7 | `copy_files()` / `copy_fs()` | Share or `dup_fd()` the file descriptor table |
| 8 | `copy_sighand()` / `copy_signal()` | Share or copy signal handlers and blocked/pending masks |
| 9 | `alloc_pid()` | Allocate PID from the namespace's IDR (integer allocator); set `p->pid`, `p->tgid` |
| 10 | `copy_namespaces()` | Copy or share `nsproxy` (PID ns, net ns, mount ns, IPC ns, UTS ns, time ns) |
| 11 | `cgroup_fork()` / `cgroup_post_fork()` | Attach child to parent's cgroup hierarchy (CPU shares, memory limits, I/O weights) |
| 12 | `copy_process()` tree linkage | Set `p->real_parent = current`; `list_add` to `parent->children`; `attach_pid()` into PID hash |
| 13 | `list_add_tail_rcu(&p->tasks, ...)` | Make process visible in global task list (used by `/proc`, `ps`, `for_each_process`) |
| 14 | `trace_sched_process_fork()` | Fire tracepoint (for perf, ftrace, eBPF observability) |
| 15 | `wake_up_new_task(p)` | `p->state = TASK_RUNNING`; `activate_task()` → `enqueue_task_fair()` places `p` on the CFS run queue of the selected CPU |
| 16 | Scheduler selects child → `__switch_to_asm` | Saves prev's callee-saved regs; switches RSP to child's kernel stack; restores child's regs; "returns" to `ret_from_fork` → `ret_to_user` |

---

## Part 7 — Multi-Process Architecture

### 7.1 What It Is

An application structured as **multiple cooperating processes** rather than a single monolithic one. Each process has its own `mm_struct`, PID, and kernel-managed resources. Their interaction produces the emergent application behavior visible to the user.

Examples: Chrome (one process per tab), PostgreSQL (postmaster + worker processes), Nginx (master + worker processes), SSH (sshd spawning per-connection children).

### 7.2 Spawning: `fork()` vs. `posix_spawn()` vs. `clone()`

**`fork()` + `execve()`:** The traditional Unix pattern. The parent calls `fork()` (creating an identical child with CoW pages), the child calls `execve()` (replacing its image). Cost: duplicating the parent's page table (O(n) in VMA count); actual physical pages are not copied until modified.

**`posix_spawn()`:** A POSIX function that internally calls `fork()` + `execve()` but is specified to behave correctly in multi-threaded processes where raw `fork()` is dangerous (only the calling thread survives in the child, but all mutexes remain locked). Some implementations use `vfork()` or `clone(CLONE_VFORK)` internally for efficiency.

**`clone()` with `CLONE_VM`:** Creates a thread (shared address space). Used by `pthread_create()` via glibc. No page table duplication at all; same `pgd` in CR3.

### 7.3 Inter-Process Communication (IPC)

Isolated address spaces mean direct memory sharing is not possible by default. The kernel provides:

| Mechanism | Kernel Object | Use Case |
|---|---|---|
| `pipe()` / `pipe2()` | `struct pipe_inode_info` | Unidirectional byte stream between related processes |
| Unix domain socket | `struct unix_sock` | Bidirectional, between any processes on same host |
| `mmap(MAP_SHARED)` | Shared VMA in both `mm_struct`s, same physical pages | High-throughput shared memory |
| `shmget()` / POSIX `shm_open()` | `struct shmid_kernel` | SysV / POSIX shared memory |
| `msgget()` | `struct msg_queue` | Message queues |
| `semget()` | `struct sem_array` | Semaphores for synchronization |
| `eventfd()` | `struct eventfd_ctx` | Lightweight event notification |
| `signalfd()` / `kill()` | Signal delivery via `task_struct->pending` | Async notification |

---

## Summary: Mental Model

```
task_struct
├── Identity
│   ├── pid          (kernel TID)
│   ├── tgid         (userspace PID)
│   └── comm         (name)
├── State            (RUNNING / INTERRUPTIBLE / UNINTERRUPTIBLE / ZOMBIE / STOPPED)
├── Scheduling
│   ├── sched_entity se     (CFS: vruntime, load)
│   ├── policy / prio
│   └── cpus_allowed
├── CPU State
│   ├── thread_struct.sp    → kernel stack pointer (restored by __switch_to_asm)
│   ├── thread_struct.fpu   → FPU/SIMD state
│   └── pt_regs @ top of kernel stack
│       ├── rip             → user-mode instruction pointer
│       ├── rsp             → user-mode stack pointer
│       ├── rax             → return value (0 in child after fork)
│       └── [all other GPRs, rflags, segment selectors]
├── Memory
│   └── mm_struct
│       ├── pgd             → loaded into CR3 on context switch
│       ├── VMAs            → memory-mapped regions (text, heap, stack, mmap)
│       └── mm_users        → reference count (threads sharing this space)
├── Resources
│   ├── files_struct        → file descriptor table [0=stdin, 1=stdout, 2=stderr, ...]
│   ├── signal_struct       → signal state
│   └── sighand_struct      → signal handler function pointers
├── Security
│   └── cred                → uid, gid, euid, capabilities
├── Namespaces
│   └── nsproxy             → PID ns, net ns, mount ns, IPC ns, UTS ns
├── Cgroups
│   └── css_set             → resource limits (CPU, memory, I/O)
└── Tree
    ├── real_parent / parent
    ├── children (list_head)
    └── tasks (global doubly-linked list)
```

---

*Sources: `kernel/fork.c`, `include/linux/sched.h`, `arch/x86/include/asm/processor.h`, `arch/x86/include/asm/ptrace.h`, `arch/x86/kernel/process_64.c`, `arch/x86/entry/entry_64.S`, `fs/exec.c`, `include/linux/mm_types.h`. Kernel version: mainline 6.x.*

Hence process creation must go thorugh a system call. System calls are a Kernel provided API interface for user process to do things and are exposed as C/C++ functions that programmers can use: : 
1. Process Control : create, terminate, load/execute, get/set process attributes, wait event, signal event, allocate memory, free memory : fork(), exit(), wait()
2. Device Management : request device, release device, read / write / reposition, get / set device attributes, logically attach / detach devices. ioctl(), read(), write()
3. File Management : CRUD, get / set file attributes. open(), read(), write(), close()
4. Comms : create / delete comms, send / receive messages, attach / detach remote devices
5. Info Maint : time, date getpid()
6. Protections : get file security  chmod(), chown()

These are functions that are invoked in the user space but executed in the kernel space, once done, exeuction returns back to user mode.




Fork in a unix-like OS (macOS, Linux distros) first clones the parent process completely (deep copy with a pid and page table exception), including the current execution state and CPU state.

The entire address space is duplicated - text section, data section, stack and heap.

The process context (almost) is duplicates as well and this includes the cpu state (instruction pointer, stack pointer, register states, etc). This means that the moment fork is called there are two processes running from the exact next instruction.   

The next system call is execv(path, argv) where path is the path of the executable and argv is an array of c strings that give the flags and arguments.
Execv replaces or resets the calling process entirely. SInce the program counter is reset, it now again points to the first instruction in the program.

How do we ensure that only the child process calls execv?
fork returns the pid of the calling process, but for a child process it always returns 0. This is how a process can know if it is a child or not.

Posix compliant systems provide a system call called posix_spawn() that internally uses fork and makes the child process called execve.



# Memory
We as CS students operate at a hihger level of abstraction where we can see memory as an array of addresses where 1 byte of information can be stored.
Any data or instruction that the CPU is going to execute needs to be loaded into main memory.

Concurrency is where multiple programs share cpu time with scheduling algoritms determining the fair CPU usage.

Learn about paging and memory management with MMU.

Also learn about how program executes, with the diefferent memory sections and cpu states.

![Memory Layout of Process](../figures/process_memory_layout.png)

this is also called the "address space". The text section is fixed as it contains the executable instruction of the program. Its the heap and stack which keep on growing and shrinking during a processe's execution.

For compiled languages like C, C++ and Rust, the compilation process produces an executable. 
For interpreted languages like Python and Javascript, its the python.exe interpreter that runs, while the .py source code is simply a text file stored in the heap of the interpreter.

We can make the difference between compiler and assembler here a bit more clear.

Two serve concurrent user programs, the OS has a scheduler and dispatcher that allow 1 process to execute on a CPU at once.
Concurreny requires saving the cpu execution state or context (all registers, flags, IP, Stack pointer etc) such that the state can be restored. This is also called context switch. 

In essence a process is a hetereogenous collection called a context (isolated from other processes) that comprises of:
1. CPU execution state
2. Address Space (memory layout)
3. Open files
4. I/O devices


This is why we call it context switching.


THe OS uses Process Control Block to keep track of every running process.
1. pid
2. state (running, new, terminated, ready, waiting)
3. program_counter (is this the same as instruction pointer?)
4. list of general purpose registers
5. stack pointer?
6. flags
7. index registers
8. memory_limits (address space start and end, maybe page tables?)
9. io_devices
10. open_files
11. pointer to parent's PCB
12. pointer to children's PCB

Stuff 3-7 is the cpu state of the process.
This PCB object is what captures the entire running state of the program with accounting metadate that can be scheduled.