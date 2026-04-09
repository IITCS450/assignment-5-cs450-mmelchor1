# Assignment 5 Results: User-Level Threads

## 1. Context-Switching Approach
In this assignment, I implemented user-level context switching in `uswtch.S` for the x86 architecture. Since this is a cooperative threading library, the switch only occurs when a thread voluntarily yields control or exits.

The process follows these steps:
* **Saving State:** When `uswtch` is called, it saves the callee-saved registers (`%ebp`, `%ebx`, `%esi`, and `%edi`) onto the current thread's stack.
* **Stack Swapping:** The current stack pointer (`%esp`) is stored in the `old` context structure, and the processor's `%esp` is updated to the `new` thread's saved stack pointer.
* **Restoring State:** The saved registers are popped off the new thread's stack, restoring its previous execution state.
* **Transition:** The `ret` instruction jumps to the address at the top of the new stack (either the return address of its last `thread_yield` or the entry point of the thread stub).

## 2. Cooperative Scheduling & Mutex
The scheduler uses a **Round-Robin** approach, iterating through the thread table to find the next thread in the `RUNNABLE` state.

Because the environment is cooperative (no timer interrupts to force preemption), the `umutex` implementation is simplified:
* **`mutex_lock`**: Uses a `while` loop to check the lock status. If the lock is held, it immediately calls `thread_yield()` to allow other threads to progress.
* **`mutex_unlock`**: Simply sets the lock variable to 0. 
* Atomic instructions like `xchg` are not strictly required here because context switches only happen at explicit yield points, ensuring the lock check-and-set operation is effectively atomic.

## 3. Limitations
* **Maximum Threads:** The library is configured to support a maximum of **8 threads**.
* **Stack Size:** Each thread has a fixed-size stack (4KB). This is sufficient for the producer-consumer demo but could lead to overflows with deep recursion or large local variables.
* **No Preemption:** A "greedy" thread that never yields or performs I/O will block all other threads indefinitely.
* **Single Core:** This implementation is designed for single-processor execution within a single xv6 process.

## 4. Verification
The implementation was verified using the `test_pc` program.
* **Producers:** Two producer threads each generated 200 items (400 total).
* **Consumer:** A single consumer thread successfully processed all 400 items.
* **Output:**