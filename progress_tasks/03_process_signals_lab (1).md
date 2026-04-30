

**Lab 03**

**Process Signals**

Topic:  Process Signals

Level:   Intermediate

Time:    50 – 65 minutes

# **Mission Brief**

| 🎬  Scenario *The NovaByte portal has been acting up again — a background worker is hanging and burning CPU.* *Your team lead needs you to communicate with the process directly: pause it, reload its config without downtime, and finally shut it down cleanly.* *Every action must be done with signals. No process manager. No restart scripts. Just you and kill.* |
| :---- |

Signals are the operating system's native language for communicating with processes. This lab teaches you to speak it fluently — from the keyboard shortcut that pauses a process to the uncatchable kill that forces it out of existence.

## **What You Will Learn**

* Read and decode the complete signal table

* Send SIGTERM, SIGKILL, SIGSTOP, SIGCONT, and SIGHUP

* Understand why SIGKILL cannot be caught or ignored

* Use kill, pkill, and keyboard shortcuts correctly

* Write a trap handler in a shell script

| Task 3.1  Explore the Signal Table |
| :---- |

| 🎬  Scenario *Before sending any signals, you need to know what weapons you have available.* |
| :---- |

List every signal available on your system:

| $ kill \-l |
| :---- |

| OUTPUT | HUP INT QUIT ILL TRAP ABRT BUS FPE KILL USR1 SEGV USR2 PIPE ALRM TERM STKFLT CHLD CONT STOP TSTP TTIN TTOU URG XCPU XFSZ VTALRM PROF WINCH POLL PWR SYS |
| :---- | :---- |

| 💡  Key signals every sysadmin must know:     SIGHUP  (1)  — Reload config without restarting     SIGINT  (2)  — Ctrl \+ C interrupt     SIGKILL (9)  — Force kill (cannot be caught)        SIGTERM (15) — Graceful shutdown request     SIGSTOP (19) — Pause (cannot be caught)             SIGCONT (18) — Resume paused process |
| :---- |

| Task 3.2  Send SIGTERM — the Polite Request to Stop |
| :---- |

| 🎬  Scenario *A worker process has completed its batch but did not exit cleanly. Request a graceful shutdown first.* |
| :---- |

Start a process to work with:

| $ sleep 400 & |
| :---- |

| OUTPUT | \[1\] 4201 |
| :---- | :---- |

Send the default termination signal (SIGTERM \= signal 15):

| $ kill 4201 |
| :---- |

Verify it is gone:

| $ ps \-p 4201 |
| :---- |

| OUTPUT |   PID TTY          TIME CMD (no output — process is gone) |
| :---- | :---- |

| 💡  kill with no signal flag defaults to SIGTERM (15). The process receives it and can choose how to respond.     Well-written programs use SIGTERM to flush buffers, close connections, and exit cleanly. |
| :---- |

| Task 3.3  Send SIGKILL — the Uncatchable Force |
| :---- |

| 🎬  Scenario *SIGTERM was ignored — the process is frozen and unresponsive. Escalate to SIGKILL.* |
| :---- |

| $ sleep 400 & |
| :---- |

| OUTPUT | \[1\] 4280 |
| :---- | :---- |

| $ kill \-9 4280 |
| :---- |

| OUTPUT | \[1\]+  Killed                  sleep 400 |
| :---- | :---- |

| 💡  SIGKILL (9) is handled entirely by the kernel — the process never sees it.     It cannot be caught, blocked, or ignored. The kernel removes the process immediately.     Always try SIGTERM first. SIGKILL leaves no chance for cleanup. |
| :---- |

| Task 3.4  Freeze and Unfreeze with SIGSTOP / SIGCONT |
| :---- |

| 🎬  Scenario *You need to pause a CPU-hungry process temporarily to let an urgent task get CPU time, then resume it.* |
| :---- |

| $ sleep 600 & |
| :---- |

| OUTPUT | \[1\] 4310 |
| :---- | :---- |

Pause the process with SIGSTOP:

| $ kill \-SIGSTOP 4310 |
| :---- |

Check its state — it should now show T (stopped):

| $ ps \-o pid,stat,comm \-p 4310 |
| :---- |

| OUTPUT |   PID STAT COMMAND   4310 T    sleep |
| :---- | :---- |

Resume it with SIGCONT:

| $ kill \-SIGCONT 4310 |
| :---- |

| $ ps \-o pid,stat,comm \-p 4310 |
| :---- |

| OUTPUT |   PID STAT COMMAND   4310 S    sleep |
| :---- | :---- |

| Task 3.5  Use pkill to Signal by Name |
| :---- |

| 🎬  Scenario *Three identical worker processes are running. You need to shut them all down with one command, by name.* |
| :---- |

| $ sleep 700 & $ sleep 701 & $ sleep 702 & |
| :---- |

| OUTPUT | \[1\] 4400 \[2\] 4401 \[3\] 4402 |
| :---- | :---- |

| $ pkill sleep |
| :---- |

| OUTPUT | \[1\]   Terminated              sleep 700 \[2\]-  Terminated              sleep 701 \[3\]+  Terminated              sleep 702 |
| :---- | :---- |

| 💡  pkill matches by pattern, not exact name. pkill slee would also work here.     killall requires the exact process name. pkill is more flexible for scripting. |
| :---- |

| Task 3.6  Trap a Signal in a Shell Script (Challenge) |
| :---- |

| 🎬  Scenario *The team wants a graceful shutdown handler in the new worker script. When SIGTERM arrives, it should log a message and clean up before exiting.* |
| :---- |

Create this script:

| $ cat \> worker.sh \<\< 'EOF' \#\!/bin/bash trap "echo '\[Worker\] SIGTERM received. Cleaning up...'; exit 0" SIGTERM echo "\[Worker\] Started. PID: 65" while true; do sleep 1; done EOF |
| :---- |

| $ chmod \+x worker.sh $ ./worker.sh & |
| :---- |

| OUTPUT | \[1\] 4501 \[Worker\] Started. PID: 4501 |
| :---- | :---- |

| $ kill 4501 |
| :---- |

| OUTPUT | \[Worker\] SIGTERM received. Cleaning up... \[1\]+  Done                    ./worker.sh |
| :---- | :---- |

| 💡  The trap command registers a handler function for a given signal.     Without trap, SIGTERM causes immediate exit. With trap, you control what happens first. |
| :---- |

## **Reflection Questions**

1. Why does kill \-9 always work while kill sometimes does not?

2. A process catches SIGTERM and delays shutdown for 30 seconds. Is this valid behaviour? When is it a problem?

3. What is the difference between SIGSTOP and SIGTSTP? Which one can a process ignore?

4. Your script receives SIGTERM mid-way through writing a file. Without a trap handler, what could go wrong?

5. In what real-world scenario would you use SIGHUP instead of restarting a service?

