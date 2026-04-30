

**Lab 06**

**Process Forking**

Topic:  Process Forking

Level:   Intermediate

Time:    55 – 70 minutes

# **Mission Brief**

| 🎬  Scenario *A production incident report is on your desk: 'After the deployment, the server is running 40 more processes than expected.'* *Nobody knows where they came from. Your task is to trace the family tree — find which parent is spawning them,* *understand how forking works at the OS level, and write a test to prove you can reproduce and observe the behaviour yourself.* |
| :---- |

Forking is the fundamental mechanism by which Linux creates new processes. Every process on your system — from your shell to a web server worker — was created by a fork. Understanding it means understanding how the entire process tree works.

## **What You Will Learn**

* Understand how fork() creates parent-child relationships

* Trace PID and PPID through a process tree

* Build and observe a fork in a shell script

* Understand orphan and zombie process creation

* Demonstrate fork() at the code level using Python

| Task 6.1  Observe Your Shell's Own Fork Lineage |
| :---- |

| 🎬  Scenario *Every command you run in a terminal was forked. Start by observing the chain that already exists around you.* |
| :---- |

Check your current shell's PID and its parent:

| $ echo "My PID: $$" $ echo "My Parent PID: $PPID" |
| :---- |

| OUTPUT | My PID: 7101 My Parent PID: 6980 |
| :---- | :---- |

Open a subshell and observe the fork:

| $ bash \-c 'echo "Child PID: $$"; echo "Child PPID: $PPID"' |
| :---- |

| OUTPUT | Child PID: 7142 Child PPID: 7101 |
| :---- | :---- |

| 💡  The subshell's PPID matches your current shell's PID.     When bash ran the \-c command, it called fork() to create a child process, then exec() to replace it with the subshell. |
| :---- |

| Task 6.2  View the Process Tree |
| :---- |

| 🎬  Scenario *The incident report mentions a 'tree of unexpected processes.' Learn to read the tree before investigating.* |
| :---- |

| $ pstree \-p $$ |
| :---- |

| OUTPUT | bash(7101)───pstree(7155) |
| :---- | :---- |

View the broader system tree from PID 1:

| $ pstree \-p 1 | head \-20 |
| :---- |

| OUTPUT | systemd(1)─┬─accounts-daemon(412)─┬─{gdbus}(413)            │                      └─{gmain}(414)            ├─cron(520)            ├─nginx(1842)─┬─nginx(1843)            │             └─nginx(1844)            └─sshd(2200)───sshd(2301)───bash(2310)───pstree(7155) |
| :---- | :---- |

| 💡  nginx(1842) is the master. nginx(1843) and nginx(1844) are workers it forked.     This is the classic master-worker pattern. The master handles config; workers handle requests. |
| :---- |

| Task 6.3  Fork Multiple Children in a Shell Script |
| :---- |

| 🎬  Scenario *You need to reproduce the 'extra processes' scenario in a controlled way — spawn children, watch them, clean them up.* |
| :---- |

Create and run a forking script:

| $ cat \> fork\_demo.sh \<\< 'EOF' \#\!/bin/bash echo "\[Parent\] PID: $$, PPID: $PPID" (echo "\[Child 1\] PID: $$, PPID: $PPID"; sleep 5\) & (echo "\[Child 2\] PID: $$, PPID: $PPID"; sleep 5\) & (echo "\[Child 3\] PID: $$, PPID: $PPID"; sleep 5\) & echo "\[Parent\] Waiting for all children..." wait echo "\[Parent\] All children finished." EOF $ chmod \+x fork\_demo.sh $ ./fork\_demo.sh |
| :---- |

| OUTPUT | \[Parent\] PID: 7200, PPID: 7101 \[Parent\] Waiting for all children... \[Child 1\] PID: 7201, PPID: 7200 \[Child 2\] PID: 7202, PPID: 7200 \[Child 3\] PID: 7203, PPID: 7200 \[Parent\] All children finished. |
| :---- | :---- |

| 💡  Each child's PPID is 7200 — the parent's PID. This is the fork relationship.     wait tells the parent to block until all its children have exited.     This prevents zombie processes from forming. |
| :---- |

| Task 6.4  Create and Observe an Orphan Process |
| :---- |

| 🎬  Scenario *An orphan process is one whose parent has died. The kernel automatically re-parents it to PID 1 (systemd).* *Reproduce this and confirm the re-parenting.* |
| :---- |

| $ cat \> orphan.sh \<\< 'EOF' \#\!/bin/bash echo "\[Parent\] PID: $$" (sleep 10; echo "\[Child\] My PPID is now: $PPID") & echo "\[Parent\] Exiting immediately." exit 0 EOF $ chmod \+x orphan.sh $ ./orphan.sh |
| :---- |

| OUTPUT | \[Parent\] PID: 7300 \[Parent\] Exiting immediately. |
| :---- | :---- |

Within 10 seconds, check the child's new parent:

| $ ps \-o pid,ppid,comm | grep sleep |
| :---- |

| OUTPUT |   PID  PPID COMMAND  7301     1 sleep |
| :---- | :---- |

| 💡  The child's PPID changed from 7300 (dead parent) to 1 (systemd).     The kernel does this automatically — no process is ever truly parentless.     Orphans are not a problem in themselves — zombies are. |
| :---- |

| Task 6.5  Demonstrate fork() in Python |
| :---- |

| 🎬  Scenario *The development team wants to understand fork at the code level. Write a Python demonstration and run it.* |
| :---- |

| $ cat \> fork\_python.py \<\< 'EOF' import os print(f'\[Before fork\] PID={os.getpid()}') pid \= os.fork() if pid \== 0:     print(f'\[Child\]  PID={os.getpid()}, PPID={os.getppid()}')     os.\_exit(0) else:     print(f'\[Parent\] PID={os.getpid()}, Child PID={pid}')     os.wait()   \# Prevent zombie     print('\[Parent\] Child collected. No zombie.') EOF $ python3 fork\_python.py |
| :---- |

| OUTPUT | \[Before fork\] PID=7400 \[Parent\] PID=7400, Child PID=7401 \[Child\]  PID=7401, PPID=7400 \[Parent\] Child collected. No zombie. |
| :---- | :---- |

| 💡  os.fork() returns twice:     In the parent process → returns the child's PID (a positive number)     In the child process  → returns 0     This is how both parent and child can exist in the same code with different behaviour. |
| :---- |

## **Reflection Questions**

1. After fork(), both parent and child are running the same code. How does each process know which role it is?

2. A server is found to have 200 zombie processes. What does this tell you about how the parent process was written?

3. What is the difference between an orphan process and a zombie process in terms of resource usage and risk?

4. Why does the kernel set the PPID of an orphan to 1 (systemd) instead of just letting it be parentless?

5. In the Python fork example, what happens if you remove the os.wait() call? What would you observe in ps?

