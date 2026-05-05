# Assignment 3 - Complete Documentation

**Student Name**: [atheer mohammed alotaibi]  
**Student ID**: [445052328]  
**Date Submitted**: [6/5/2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [May 5, 2026, 02:00 PM]
**What I implemented**:  Initial structure of SharedResources and adding ReentrantLock for each counter

**Challenges encountered**: Deciding between one global lock or multiple specific locks.

**How I solved it**:  Implemented Fine-grained locking to allow different threads to update different counters simultaneously without blocking each other

**Testing approach**: Printed lock hash codes to ensure each resource used a unique lock instance.

**Time spent**:  1 hour

---

### Entry 2 - [May 5, 2026, 05:30 PM]
**What I implemented**:  CPU access control using a Semaphore.

**Challenges encountered**: Ensuring that only one process "executes" at a time while others wait in the queue.

**How I solved it**:  Set cpuSemaphore permits to 1 and wrapped the execution logic in a try-finally block to guarantee the permit is released.

**Testing approach**:  Verified terminal output to ensure "executing quantum" messages never overlapped between different processes.

**Time spent**:  1.5 hours.

---

### Entry 3 - [May 5, 2026, 09:00 PM]
**What I implemented**:  Thread-safe executionLog and final UI formatting.

**Challenges encountered**: Risk of ConcurrentModificationException when multiple threads add entries to the ArrayList.

**How I solved it**: Created a logLock specifically for the executionLog list operations.

**Testing approach**: Ran the simulation with the maximum number of processes (20+) to stress-test concurrent list access.

**Time spent**:  1 hour.

---

### Entry 4 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

### Entry 5 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[Counter Variables:The contextSwitchCount is a shared integer. Without synchronization, two threads might perform count++ (read-modify-write) simultaneously, leading to "Lost Updates" where the final count is lower than the actual events.
 * Execution Log: The ArrayList is not thread-safe. Concurrent calls to .add() can lead to data corruption or a ConcurrentModificationException, causing the simulation to crash or lose log entries.]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[ ReentrantLock: Acts as a Mutex (Mutual Exclusion). I used it for the counters (e.g., contextSwitchLock) because only one thread should modify the value at a time to ensure atomicity.
 * Semaphore: Manages permits. I used cpuSemaphore with 1 permit to represent a single CPU core. While a Mutex is owned by a thread, a Semaphore is better for signaling and managing a pool of identical resources (in this case, CPU time).]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[Deadlock occurs when two or more threads are blocked forever, each waiting for a resource held by the other.
 * Prevention 1 (Lock Ordering): I ensured that if multiple locks were needed, they would always be acquired in the same order.
 * Prevention 2 (Timed/Safe Release): I used try-finally blocks. By placing .unlock() and .release() in the finally section, I ensured resources are freed even if an exception occurs, preventing other threads from being blocked indefinitely.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[I chose Fine-grained locking by using separate locks for each counter.
 * Why: Since the contextSwitchCount, completedProcessCount, and totalWaitingTime are logically independent, there is no reason for a thread updating one to block a thread updating another.
 * Trade-offs: Coarse-grained locking (one lock for all) is easier to implement but creates a bottleneck. Fine-grained locking increases complexity but significantly improves concurrency and performance in high-load scenarios.]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**:  contextSwitchCount, completedProcessCount, totalWaitingTime.

**Why they need protection**: 

**Synchronization mechanism used**:  ReentrantLock

**Code snippet**:
```java
   contextSwitchLock.lock();
   try { contextSwitchCount++; }
   finally { contextSwitchLock.unlock(); }
```

**Justification**:  These are shared states. The lock ensures the "read-increment-write" cycle is atomic.

---

### Critical Section #2: Execution Log

**What resource**:  List<String> executionLog.

**Why it needs protection**: 

**Synchronization mechanism used**:  ReentrantLock (logLock).

**Code snippet**:
```java
//    logLock.lock();
   try { executionLog.add(message); }
   finally { logLock.unlock(); }
```

**Justification**: Protects the internal structure of the ArrayList from concurrent modification.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To simulate exclusive CPU execution.

**Number of permits and why**: 

**Where implemented**: 1

**Code snippet**:
```java
//    SharedResources.cpuSemaphore.acquire();
   try { // execute quantum... }
   finally { SharedResources.cpuSemaphore.release(); }
   
   

```

**Effect on program behavior**:  Only one thread can enter the "running" state; others must wait in the semaphore's queue.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**:  Running the program multiple times with the same seed (Student ID) to verify that synchronization prevents non-deterministic results.
**Testing procedure**: 
```bash
#  1. Set studentID = 445052328.
 2. Executed the program 5 consecutive times.
 3. Compared the "Total Context Switches" and "Average Waiting Time" across all runs.
```

**Results**: 
(Run 1-5: All runs produced exactly the same results (e.g., Total Context Switches and Average Waiting Time remained constant).
 * Consistency: 100%.)

**Why synchronization is necessary**: 
(Without synchronization, the contextSwitchCount++ operation (which is not atomic) could lead to "Lost Updates". If two threads finish their quantum at the exact same time, they might read the same counter value, increment it, and write it back, resulting in the counter being incremented by 1 instead of 2. Even if not observed in every run, synchronization guarantees this will *never* happen.)

**Conclusion**: 

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException within the executionLog.

**Testing procedure**:  1. Increased the number of processes to the maximum range (20+).
 2. Rapidly added log entries from multiple threads using SharedResources.logExecution().
 3. Observed if the program crashed during the execution or while printing the summary.

**Results**:  * Errors: Zero ConcurrentModificationException encountered.
 * Log Integrity: All log entries were successfully added and the executionLog.size() matched the expected number of events.

**What this proves**: This proves that the logLock successfully protected the ArrayList. Since ArrayList is not thread-safe, the lock ensured that only one thread could modify the list's internal array at a time.

---

### Test 3: Correctness Verification
**What I tested**: Verifying that the final counters match the actual process lifecycle.

**Expected values**: * Completed Processes: Should exactly match numProcesses.
 * Remaining Time: Every process must reach 0ms.

**Actual values**: * Completed Processes: Matches the number of threads created.
 * Wait Time Calculation: The formula (completionTime - creationTime) - burstTime correctly yielded non-negative values for all processes.

**Analysis**: The synchronization using the cpuSemaphore ensured that completionTime was only recorded after the process actually finished its final burst, leading to accurate statistics in the "Process Summary Table".

---

### Test 4: Different Scenarios
**Scenario tested**: [Short Time Quantum vs. Long Time Quantum.

**Purpose**: To see how the synchronization handles high-frequency context switching.

**Results**:  With a Short Quantum (e.g., 500ms): The number of contextSwitchCount increased significantly. The contextSwitchLock handled the high contention without any race conditions.
 * With a Long Quantum: Processes finished in fewer turns, and the cpuSemaphore correctly managed the transition between the run() method and the runToCompletion() method for the last process.

**What I learned**: Synchronization is most critical when the "Time Quantum" is small, as threads are constantly yielding and acquiring the CPU, which increases the frequency of shared resource access.

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[What I learned about synchronization:
Working on this simulation taught me that synchronization is the backbone of any multitasking operating system. I learned that even simple operations like count++ are dangerous in a multi-threaded environment because they are not "atomic." The most challenging part was ensuring that the cpuSemaphore was always released in the finally block; otherwise, the entire CPU would "hang," and no other process could execute.I also realized that using fine-grained locks (separate locks for each counter) is much better for performance than a single global lock.

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: Database Management Systems (DBMS). When multiple users try to update the same record (like a bank balance), locks ensure that one transaction finishes before the next one starts to prevent money from "disappearing."

**Example 2**:  Print Spoolers. A printer is a shared resource. A semaphore ensures that only one document is printed at a time, preventing lines of text from different files from being mixed on the same page.

---

### How I would explain synchronization to others:

[Imagine a shared classroom with only one whiteboard marker (the CPU). Even if there are 20 students (threads) who want to write, only the one holding the marker can write. Synchronization is the set of rules—like a "sign-up sheet" (the Queue) and the "handing over of the marker" (the Semaphore)—that ensures students don't fight over the marker or write over each other’s work. It turns chaos into an organized line.]

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/atheer-mohammed-445/OS-Assignment3-atheer-mohammed-445.git

**Number of commits**: 10 commits

**Commit messages**: 
1. sit my student id:445052328
2. Add Semaphore and ReentrantLock imports
3. Implement synchronization mechanisms in SharedResources
4. Introduce cpuSemaphore for CPU access control

---

## Summary

**Total time spent on assignment**: 
Around 8 hours
**Key takeaways**: 
1. Concurrency Control: Learned how to effectively use ReentrantLock to prevent race conditions in shared counters and logs.
2. Resource Management: Understood the role of Semaphores in simulating hardware constraints like CPU access.
3. Robustness: Realized that using try-finally blocks is essential to prevent deadlocks by ensuring locks are always released.

**Most challenging aspect**: The most challenging part was implementing fine-grained locking. I had to ensure that using multiple separate locks for each counter didn't lead to complex dependencies, while still maintaining the thread-safety of the executionLog during high-frequency context switching.

**What I'm most proud of**: 
I am most proud of achieving a fully synchronized simulation where the results are 100% consistent across multiple runs. Specifically, the integration of the cpuSemaphore within the runToCompletion method ensures the scheduler remains stable even when only one process is left.
---

**End of Documentation**
