

**Lab 04**

**Killing Processes**

Topic:  Killing Processes

Level:   Intermediate

Time:    45 – 60 minutes

# **Mission Brief**

| 🎬  Scenario *Three rogue worker processes have broken loose on NovaByte's staging server. One is ignoring shutdown commands.* *One was killed but left zombie children behind. You need to handle all three — gracefully, forcefully, and by name.* *The on-call pager is watching. Get them cleaned up before the automated health check fires.* |
| :---- |

Killing a process is not just about force — it's about choosing the right method for the situation. This lab teaches you the full escalation path from polite request to kernel-level forced termination.

## **What You Will Learn**

* Terminate processes by PID, job number, and name

* Understand the escalation path: SIGTERM → SIGKILL

* Use kill, pkill, and killall with correct flags

* Identify and clean up zombie processes

* Kill multiple processes in a single command

| Task 4.1  Kill by PID — The Basic Termination |
| :---- |

| 🎬  Scenario *Worker 1 is still responding to signals. Use a clean, graceful shutdown.* |
| :---- |

Start it:

| $ sleep 500 & |
| :---- |

| OUTPUT | \[1\] 5100 |
| :---- | :---- |

Send SIGTERM:

| $ kill 5100 |
| :---- |

Confirm termination:

| $ ps \-p 5100 |
| :---- |

| OUTPUT |   PID TTY          TIME CMD (no output — process no longer exists) |
| :---- | :---- |

| 💡  kill \<PID\> sends SIGTERM by default. The process gets a chance to clean up before exiting.     This is always the right first step. Jumping straight to kill \-9 is bad practice. |
| :---- |

| Task 4.2  Kill Multiple Processes by PID in One Command |
| :---- |

| 🎬  Scenario *Workers 2, 3, and 4 need to go. Three separate kill commands would be slow. Do it in one.* |
| :---- |

| $ sleep 501 & sleep 502 & sleep 503 & |
| :---- |

| OUTPUT | \[1\] 5201 \[2\] 5202 \[3\] 5203 |
| :---- | :---- |

Kill all three at once:

| $ kill 5201 5202 5203 |
| :---- |

| OUTPUT | \[1\]   Terminated              sleep 501 \[2\]-  Terminated              sleep 502 \[3\]+  Terminated              sleep 503 |
| :---- | :---- |

| Task 4.3  Force Kill with \-9 When SIGTERM Fails |
| :---- |

| 🎬  Scenario *Worker 5 is misbehaving — it has a custom SIGTERM handler that deliberately ignores shutdown.* *You've already tried the graceful approach. Escalate.* |
| :---- |

| $ sleep 600 & |
| :---- |

| OUTPUT | \[1\] 5301 |
| :---- | :---- |

Attempt graceful kill (assume it was ignored):

| $ kill 5301 |
| :---- |

Process is still there. Escalate to SIGKILL:

| $ kill \-9 5301 |
| :---- |

| OUTPUT | \[1\]+  Killed                  sleep 600 |
| :---- | :---- |

| 💡  SIGKILL bypasses any in-process handlers. The kernel removes the process immediately.     Downside: no cleanup. Open files may be left dirty, database transactions incomplete. |
| :---- |

| Task 4.4  Kill by Name with killall |
| :---- |

| 🎬  Scenario *You need to terminate all sleep processes — there are several and you don't want to track PIDs manually.* |
| :---- |

| $ sleep 700 & sleep 701 & sleep 702 & |
| :---- |

Kill all by exact name:

| $ killall sleep |
| :---- |

| OUTPUT | \[1\]   Terminated              sleep 700 \[2\]-  Terminated              sleep 701 \[3\]+  Terminated              sleep 702 |
| :---- | :---- |

| 💡  killall matches exact process names. pkill matches patterns.     killall nginx   — only matches 'nginx' exactly     pkill ngin      — matches anything containing 'ngin' |
| :---- |

| Task 4.5  Kill by Job Number |
| :---- |

| 🎬  Scenario *You've just started a process in the current terminal session and immediately realise it was wrong.* *You don't need the PID — use the job number.* |
| :---- |

| $ sleep 800 & |
| :---- |

| OUTPUT | \[1\] 5400 |
| :---- | :---- |

| $ jobs |
| :---- |

| OUTPUT | \[1\]+  Running                 sleep 800 & |
| :---- | :---- |

Kill it by job number:

| $ kill %1 |
| :---- |

| OUTPUT | \[1\]+  Terminated              sleep 800 |
| :---- | :---- |

| Task 4.6  Detect and Clean Up a Zombie Process |
| :---- |

| 🎬  Scenario *A parent process exited without collecting its child's exit status. The child is now a zombie — it appears in ps but uses no resources.* *You need to find it and understand how to resolve it.* |
| :---- |

Create a zombie (the parent exits before calling wait):

| $ cat > zombie.sh << 'EOF'
#!/bin/bash

(sleep 2; exit 0) &
ZPID=$!

echo "Child PID: $ZPID"

sleep 30   # keep parent alive longer
EOF|
| $ chmod +x zombie.sh |
| $ ./zombie.sh &|
| :---- |

| OUTPUT | \[1\] 5501 Child PID: 5502 |
| :---- | :---- |

Within 15 seconds, check for the zombie (Z in the STAT column):

| $ ps aux | grep Z |
| :---- |

| OUTPUT | student   5502  0.0  0.0      0     0 pts/0    Z+   10:01   0:00 \[sleep\] \<defunct\> |
| :---- | :---- |

| 💡  A zombie is a process that has finished but whose exit status has not been collected by its parent.     It takes no CPU or memory — only a slot in the process table.     Killing the zombie directly won't work (it's already dead). Kill the parent instead. |
| :---- |

Kill the parent to let the OS clean up the zombie:

| $ kill 5501 |
| :---- |

| OUTPUT | \[1\]+  Terminated              ./zombie.sh |
| :---- | :---- |

## **Reflection Questions**

1. When should you use SIGTERM vs SIGKILL? Write a decision rule you would follow in production.

2. You killed a parent process but a child became a zombie. What caused this and how do you resolve it?

3. What is the difference between kill %1 and kill \<PID\>? When is each useful?

4. A script sends kill \-9 immediately without trying SIGTERM first. Why is this problematic?

5. killall failed with 'no process found' but ps shows the process running. What might be wrong?

