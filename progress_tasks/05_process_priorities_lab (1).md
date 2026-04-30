

**Lab 05**

**Process Priorities**

Topic:  Process Priorities

Level:   Intermediate

Time:    45 – 55 minutes

# **Mission Brief**

| 🎬  Scenario *NovaByte's server is running both a customer-facing API and a nightly database backup job.* *At 11 PM, the backup job kicks in and starts consuming so much CPU that the API becomes sluggish.* *You have been asked to tune the priorities: keep the API responsive and push the backup job into the background.* *Every change must be done live — no restarts, no downtime.* |
| :---- |

Linux gives every process a scheduling priority called the nice value. This lab teaches you to read it, change it, and understand why the privilege boundary exists.

## **What You Will Learn**

* Read nice values using ps and top

* Start processes with custom priority using nice

* Change running process priority using renice

* Understand the \-20 to \+19 scale and what each end means

* Understand why only root can assign negative nice values

| Task 5.1  Read the Default Priority |
| :---- |

| 🎬  Scenario *Before tuning anything, establish what priority your processes are currently running at.* |
| :---- |

| $ sleep 300 & |
| :---- |

| OUTPUT | \[1\] 6100 |
| :---- | :---- |

Check its nice value using ps:

| $ ps \-o pid,ni,comm \-p 6100 |
| :---- |

| OUTPUT |   PID  NI COMMAND  6100   0 sleep |
| :---- | :---- |

| 💡  NI \= nice value. Default is 0\.     Scale:  \-20 (highest priority, most CPU)  →  \+19 (lowest priority, least CPU)     Lower number \= scheduler favours this process more \= gets more CPU time. |
| :---- |

| Task 5.2  Start a Process with Low Priority |
| :---- |

| 🎬  Scenario *The nightly backup job should not compete with the API. Start it with a low priority from the beginning.* |
| :---- |

| $ nice \-n 15 sleep 400 & |
| :---- |

| OUTPUT | \[2\] 6201 |
| :---- | :---- |

Confirm the nice value was applied:

| $ ps \-o pid,ni,comm \-p 6201 |
| :---- |

| OUTPUT |   PID  NI COMMAND  6201  15 sleep |
| :---- | :---- |

Start another at the lowest possible priority:

| $ nice \-n 19 sleep 401 & |
| :---- |

Compare all three:

| $ ps \-o pid,ni,comm | grep sleep |
| :---- |

| OUTPUT |   PID  NI COMMAND  6100   0 sleep  6201  15 sleep  6202  19 sleep |
| :---- | :---- |

| Task 5.3  Start a High Priority Process (Root Required) |
| :---- |

| 🎬  Scenario *The API service needs elevated CPU priority. Only root can assign negative nice values — try it both ways.* |
| :---- |

Try without sudo — this should fail:

| $ nice \-n \-10 sleep 500 & |
| :---- |

| OUTPUT | nice: cannot set niceness: Permission denied |
| :---- | :---- |

Try with sudo:

| $ sudo nice \-n \-10 sleep 500 & |
| :---- |

| $ ps \-o pid,ni,comm | grep sleep |
| :---- |

| OUTPUT |   PID  NI COMMAND  6100   0 sleep  6201  15 sleep  6202  19 sleep  6310 \-10 sleep |
| :---- | :---- |

| 💡  Only root can decrease the nice value (increase priority).     This prevents any user from monopolising the CPU by setting themselves to \-20.     In production, service files set priority via systemd — not manual nice calls. |
| :---- |

| Task 5.4  Change Priority of a Running Process with renice |
| :---- |

| 🎬  Scenario *The backup job is already running at default priority and is now slowing the API. Lower its priority live.* |
| :---- |

| $ sleep 600 & |
| :---- |

| OUTPUT | \[1\] 6400 |
| :---- | :---- |

Lower its priority to \+15:

| $ renice \+15 \-p 6400 |
| :---- |

| OUTPUT | 6400 (process ID) old priority 0, new priority 15 |
| :---- | :---- |

Confirm the change:

| $ ps \-o pid,ni,comm \-p 6400 |
| :---- |

| OUTPUT |   PID  NI COMMAND  6400  15 sleep |
| :---- | :---- |

Now elevate priority (requires sudo):

| $ sudo renice \-5 \-p 6400 |
| :---- |

| OUTPUT | 6400 (process ID) old priority 15, new priority \-5 |
| :---- | :---- |

| Task 5.5  View and Change Priorities in top |
| :---- |

| 🎬  Scenario *You need a live view of all priorities and want to renice interactively without leaving top.* |
| :---- |

| $ top |
| :---- |

| OUTPUT |   PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND  6100 student   20   0    8076    528    468 S   0.0   0.0   0:00.00 sleep  6201 student   35  15    8076    528    468 S   0.0   0.0   0:00.00 sleep  6310 root       10 \-10    8076    528    468 S   0.0   0.0   0:00.00 sleep |
| :---- | :---- |

While in top, press r to renice interactively. Enter the PID and new nice value when prompted.

| 💡  PR \= kernel's actual internal priority \= 20 \+ nice value     A nice value of 0 → PR of 20\.  A nice value of \-10 → PR of 10\.     Lower PR \= higher kernel scheduling priority. |
| :---- |

## **Reflection Questions**

1. A backup process has nice value \+19 and an API has nice value 0\. How does the kernel treat them differently when both need CPU at the same time?

2. Why can any user increase their process's nice value (make it less greedy) but only root can decrease it?

3. renice applies to the target process only — not its children. Why could this be a problem for a parent process that spawns workers?

4. In a real production deployment, where would you set service priority instead of using manual renice calls?

5. You set a background job to nice \+19 but it still appears to slow the system during peak load. What else might be the bottleneck?

