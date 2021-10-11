# _

## Abstraction

### Process

#### Direct Execution Protocol

#### Restricted Operations

1. user mode
2. kernel mode
3. trap table
4. system-call number

#### Switching Between Processes

1. cooperatively via a system call
2. timer interrupt

The hardware assists the OS by providing different modes of execution.
In user mode, applications do not have full access to hardware resources.
In kernel mode, the OS has access to the full resources of the machine.
Special instructions to trap into the kernel and return-from-trap back to
user-mode programs are also provided, as well as instructions that allow
the OS to tell the hardware where the trap table resides in memory.

#### States

1. Running
2. Ready
3. Blocked

#### context switch

#### task list

#### Process Control Block

#### stack

#### heap

#### Process vs running program

#### time sharing vs scheduling policy

#### high-level

##### scheduling policy

##### Scheduling Metrics

1. turnaround time: The turnaround time of a job is defined as the time at which the job completes minus the time at which the job arrived in the system
2. average turnaround time: This problem is generally referred to as the convoy effect
3. non-preemptive scheduler vs preemptive scheduler
4. Response Time: as the time from when the job arrives in a system to the first time it is scheduled
5. Incorporating I/O

##### First In, First Out (FIFO)

##### Shortest Job First (SJF)

##### Preemptive Shortest Job First (PSJF)

##### Round Robin (Good For Response Time)

##### Summary

How can we design a scheduler that both minimizes response time for
interactive jobs while also minimizing turnaround time without a priori
knowledge of job length?

##### Multi-level Feedback Queue (MLFQ)

1. Basic Rules
    • Rule 1: If Priority(A) > Priority(B), A runs (B doesn’t).
    • Rule 2: If Priority(A) = Priority(B), A & B run in RR.

2. Attempt #1: How To Change Priority
    • Rule 3: When a job enters the system, it is placed at the highest priority (the topmost queue).
    • Rule 4a: If a job uses up an entire time slice while running, its priority is reduced (i.e., it moves down one queue).
    • Rule 4b: If a job gives up the CPU before the time slice is up, it stays at the same priority level.

3. Attempt #2: The Priority Boost
    to solve starvation

4. Attempt #3: Better Accounting

##### Scheduling: Proportional Share

Proportional-share is based around a simple concept: instead
of optimizing for turnaround or response time, a scheduler might instead
try to guarantee that each job obtain a certain percentage of CPU time

1. Basic Concept: Tickets Represent Your Share

##### Implementation

1. All you need is a good random number generator to pick the winning ticket, a data structure to track the processes of the system (e.g., a list), and the total number of tickets

    You might also be wondering: why use randomness at all? As we saw above, while randomness gets us a simple (and approximately correct) scheduler, it occasionally will not deliver the exact right proportions, especially over short time scales. For this reason, Waldspurger invented stride scheduling, a deterministic fair-share scheduler

2. stride scheduling

##### Linux Completely Fair Scheduler (CFS)

Basic Operation: Whereas most schedulers are based around the concept of a fixed time slice, CFS operates a bit differently. Its goal is simple: to fairly divide a CPU evenly among all competing processes. It does so through a simple counting-based technique known as virtual runtime (vruntime).

sched latency: CFS uses this value to determine how long one process should run before considering a switch (effectively determining its time slice but in a dynamic fashion)
min granularity: which is usually set to a value like 6 ms. CFS will never set the time slice of a process to less than this value, ensuring that not too much time is spent in scheduling overhead.

Weighting (Niceness):
    $timeSlice_{k}$ = ($weight_{k}$ / $\sum_{i=0}^{n-1} weight_i$) * schedLatency
    $vruntime_{i}$ = $vruntime_{i}$ + $weight_{0}$/$weight_{i}$ · $runtime_{i}$

### persistence

#### file descriptors
