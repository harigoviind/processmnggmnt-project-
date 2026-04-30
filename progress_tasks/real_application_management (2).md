# 🖥️ Real Application Management
## Linux Process Management — Final Project

---

| Detail | Info |
|---|---|
| **Course** | Linux Process Management |
| **Estimated Time** | 3 – 5 Hours |
| **Level** | Final Assessment |
| **Format** | Individual · Real Application · Terminal Only · No Scripts |

---

## 📋 The Idea

Everything you have learned in this course — PIDs, signals, priorities, forking, job control, /proc — exists because real software runs on real Linux systems every day.

This project is simple in concept but deep in execution:

> **Pick one real application. Run it on your Linux system. Apply everything you know.**

No dummy processes. No `dd if=/dev/zero`. A real program that real people use — a web server, a database, a media server, a development tool — and you will investigate it, stress it, control it, tune it, and keep it alive, exactly the way a professional sysadmin or developer would.

By the end, you will have a complete picture of how one real application behaves as a Linux process — and you will have manipulated every aspect of that behaviour using only your terminal.

---

## 🧰 Choose Your Application

Pick **one** application from the list below and install it on your Linux system. Every application in this list is free, open source, and installable in under 5 minutes.

---

### Option A — nginx (Web Server)

> **Best for:** Students who want to see how production web servers are managed

nginx is one of the most widely deployed web servers in the world. It runs as a master process with multiple worker children, uses SIGHUP to reload configuration, and is an excellent example of real-world process management in production software.

**Install:**

```
# Ubuntu / Debian
sudo apt install nginx

# Fedora / RHEL
sudo dnf install nginx

# Arch
sudo pacman -S nginx
```

**Start it:**

```
sudo systemctl start nginx
```

**Verify it is running:**

Open your browser and go to `http://localhost` — you should see the nginx welcome page.

---

### Option B — Apache HTTP Server

> **Best for:** Students who want to explore a multi-process web server model

Apache creates a separate child process for each connection it handles. This makes it a perfect subject for studying process forking, parent-child relationships, and signal-based management.

**Install:**

```
# Ubuntu / Debian
sudo apt install apache2

# Fedora / RHEL
sudo dnf install httpd

# Arch
sudo pacman -S apache
```

**Start it:**

```
# Ubuntu
sudo systemctl start apache2

# Fedora
sudo systemctl start httpd
```

**Verify it is running:**

Open your browser and go to `http://localhost`.

---

### Option C — Redis (In-Memory Data Store)

> **Best for:** Students who want a lightweight, single-process application

Redis is a fast, single-process application often used as a cache or message broker. Because it runs as a single process, it is excellent for studying signals, priority, /proc, and job survival without the complexity of multi-process trees.

**Install:**

```
# Ubuntu / Debian
sudo apt install redis-server

# Fedora / RHEL
sudo dnf install redis

# Arch
sudo pacman -S redis
```

**Start it:**

```
sudo systemctl start redis
```

**Verify it is running:**

```
redis-cli ping
```

You should see: `PONG`

---

### Option D — Python HTTP Server (No Install Needed)

> **Best for:** Students who want to start immediately with zero setup

Python's built-in HTTP server requires no installation and works on any system with Python 3. It is a real, working web server you can connect to from your browser.

**Start it:**

```
# Create a test directory first
mkdir ~/myserver && cd ~/myserver
echo "<h1>My Server is Running</h1>" > index.html

# Start the server
python3 -m http.server 8080
```

**Verify it is running:**

Open your browser and go to `http://localhost:8080` — you should see "My Server is Running".

> **Note:** For tasks that require SIGHUP config reload, use nginx or Apache instead, as Python's HTTP server does not implement config reloading. For all other tasks, the Python HTTP server works perfectly.

---

## ⚠️ Rules

- Work entirely in the terminal — no graphical tools
- Document every command you run and every output you observe
- Never simply state what a command does — explain what **you** observed on **your** system
- If something behaves differently than expected, document it honestly and reason through why
- Document your findings for each phase as you go, not at the end
- Do not restart your application between tasks unless a task explicitly tells you to

---

## 🗂️ How to Submit

When all 6 phases are complete, submit a document containing your findings for every phase. For each task, include the exact commands you ran and the real output from your system — not invented or paraphrased outputs.

> **Your documentation is your grade. A real output with a wrong explanation is better than a correct explanation with no output.**

---

# Phase 1 — First Contact
### *"Meet your application as a Linux process"*

> **Estimated Time:** 25 – 35 minutes  
> **Topics:** Process basics · PID · PPID · Process hierarchy · /proc · Process states · `ps` · `pgrep` · `pstree`

You have started your application. Now stop thinking of it as software and start thinking of it as a Linux process. Your first job is to find it, identify it completely, and understand exactly where it sits in the process hierarchy.

---

### 📌 Task 1.1 — Find Its Identity

Your application is running. Find its PID using at least two different methods.

> **Document in your report:**
> - The application name as it appears in the process list
> - Its PID (you will use this throughout the project — note it carefully)
> - The two methods you used to find it
> - Did both methods return identical results? Were there any differences in what each showed?

---

### 📌 Task 1.2 — Read Its Full Profile

Use `ps` to get the complete profile of your application process — not just its name and PID but every column of information available.

> **Document in your report:**
> - The full `ps` output for your process
> - Identify and explain each column: `USER`, `PID`, `%CPU`, `%MEM`, `VSZ`, `RSS`, `TTY`, `STAT`, `START`, `TIME`, `COMMAND`
> - What does the `STAT` column show for your application right now? What does that state mean?
> - What is the difference between `VSZ` (virtual memory) and `RSS` (resident memory)? Which is larger for your application and why?

---

### 📌 Task 1.3 — Map the Family Tree

Find your application's position in the full Linux process hierarchy — its parent and all of its children.

> **Document in your report:**
> - The full process tree from PID 1 down to your application and its children
> - Your application's PPID — what is the parent and why does that parent own it?
> - How many child processes did your application spawn? What are they named?
> - If your application has multiple workers or children, explain why it forks them

---

### 📌 Task 1.4 — Explore /proc

Navigate into the /proc filesystem and find your application's directory. Read at least four different files from it.

> **Document in your report:**
> - The full path to your application's /proc directory
> - The contents of `/proc/<PID>/status` — paste the first 15 lines
> - The contents of `/proc/<PID>/cmdline` — what was the exact command used to launch it?
> - The contents of `/proc/<PID>/limits` — what resource limits does your application have?
> - List the open file descriptors in `/proc/<PID>/fd/` — how many open files does it have? Can you identify what they are?

---

### 📌 Task 1.5 — Baseline Snapshot

Before you do anything else to your application, take a complete baseline snapshot of its resource usage.

> **Document in your report:**
> - Current CPU percentage
> - Current memory percentage
> - Current nice value
> - Current process state
> - Number of child processes
> - This is your baseline — you will compare against it later

---

# Phase 2 — Stress Test
### *"Push your application and watch what changes"*

> **Estimated Time:** 30 – 40 minutes  
> **Topics:** CPU monitoring · `top` · `htop` · `ps` sorting · process states · load generation · /proc live reading

A process sitting idle tells you very little. To really understand how your application behaves as a Linux process, you need to make it work. This phase puts your application under real load and observes how it responds at the process level.

---

### 📌 Task 2.1 — Generate Real Load

Create genuine activity for your application. The method depends on which application you chose:

- **nginx / Apache:** Use a browser to make many page requests, or open multiple browser tabs all pointing to your server simultaneously. If you want heavier load, install `curl` and run it repeatedly.
- **PostgreSQL:** Connect using `psql` and run a query multiple times, or open several terminal windows and connect from each simultaneously.
- **Redis:** Use `redis-cli` to SET and GET hundreds of keys in rapid succession.
- **Python HTTP server:** Open many browser tabs pointing to your server, or use `curl` to download a file repeatedly.

Keep the load running while you complete the remaining tasks in this phase.

> **Document in your report:**
> - Exactly what you did to generate load
> - Roughly how much activity you created

---

### 📌 Task 2.2 — Observe It Live in top

Open `top` while your application is under load and observe it in real time.

> **Document in your report:**
> - What position does your application appear at in the CPU-sorted list?
> - What is its CPU percentage under load compared to your baseline from Phase 1?
> - Did any child processes appear or change state under load?
> - What does the load average at the top of `top` show? What do the three numbers mean?
> - Press `1` in `top` to see per-CPU view — which CPUs is your application using?

---

### 📌 Task 2.3 — Capture the Loaded ps Output

While your application is still under load, take a `ps` snapshot sorted by CPU.

> **Document in your report:**
> - The full `ps` output sorted by CPU percentage
> - Where does your application rank among all processes?
> - If your application has multiple workers, how is CPU distributed among them?

---

### 📌 Task 2.4 — Read /proc Under Load

While load is still running, re-read your application's /proc status file and compare it to your baseline.

> **Document in your report:**
> - The updated `/proc/<PID>/status` — paste relevant lines
> - What changed compared to your Phase 1 baseline? Memory? State? Threads?
> - Is the process in `R` (running) or `S` (sleeping) state under load? Why?

---

### 📌 Task 2.5 — Stop the Load and Observe Recovery

Stop generating load and watch your application return to idle.

> **Document in your report:**
> - How quickly did CPU usage drop after you stopped the load?
> - Did the number of child processes change when load stopped?
> - Is the application's resource profile now identical to your Phase 1 baseline, or is there a difference? Explain any differences.

---

# Phase 3 — Signal Conversations
### *"Talk to your application using signals"*

> **Estimated Time:** 35 – 45 minutes  
> **Topics:** `SIGHUP` · `SIGTERM` · `SIGSTOP` · `SIGCONT` · `SIGKILL` · `kill` · `pkill` · signal numbers vs names

This is the most important phase. You will send real signals to a real running application and observe exactly what happens at the process level. Every task here involves a real effect — not simulation.

---

### 📌 Task 3.1 — Reload Without Restarting (SIGHUP)

Send `SIGHUP` to your application's main process.

> **Before you send it:**
> - Note the current PID and how long the process has been running (the `ELAPSED` column in `ps`)
> - Note the current number of child processes

> **Send SIGHUP and immediately check:**
> - Did the PID change? A changed PID means the process restarted. An unchanged PID means it reloaded in place.
> - Did the number of child processes change?
> - Did the `ELAPSED` time reset or continue?

> **Document in your report:**
> - The exact command you used to send `SIGHUP`
> - What your application did in response — describe in detail what you observed
> - What does this tell you about how your application handles `SIGHUP`?
> - Why is in-place reload valuable in production? What would have been lost with a full restart?

---

### 📌 Task 3.2 — Freeze and Unfreeze (SIGSTOP / SIGCONT)

Send `SIGSTOP` to your application's main process to freeze it, then verify it is frozen, then resume it with `SIGCONT`.

> **Document in your report:**
> - The `SIGSTOP` command you used
> - What happened to the application's `STAT` column immediately after `SIGSTOP`?
> - If your application is a web server — try to access it from a browser while it is frozen. What happens? Why?
> - How long did you leave it frozen?
> - The `SIGCONT` command you used
> - How quickly did the application recover after `SIGCONT`?
> - Was any data lost or corrupted by the freeze? How do you know?

---

### 📌 Task 3.3 — The Signal Escalation Test

Test the correct kill escalation sequence on your application — `SIGTERM` first, then (after it has restarted) `SIGKILL`.

> **Part A — SIGTERM:**
> - Send `SIGTERM` to your application's main process
> - Observe exactly what happens to the process and all its children
> - Check: did all child processes also exit, or did any become orphans or zombies?

> **Part B — Restart and SIGKILL:**
> - Restart your application
> - This time send `SIGKILL` instead of `SIGTERM`
> - Observe what happens — is the behaviour different from `SIGTERM`?
> - Check for zombie processes after `SIGKILL`

> **Document in your report:**
> - The exact commands for both signals
> - The observed behaviour difference between `SIGTERM` and `SIGKILL` for your specific application
> - What happened to child processes in each case?
> - Were there any zombie processes after either signal? If yes, explain why and how you resolved them.
> - Restart your application after this task before continuing

---

### 📌 Task 3.4 — Signal a Child, Not the Parent

If your application has child processes (workers), send `SIGTERM` to one child worker only — not the parent.

> If your application has no children (Redis, Python HTTP server), send `SIGSTOP` to the main process, then immediately send `SIGCONT` — and document what both signals do in sequence.

> **Document in your report:**
> - The child PID you targeted and why you chose it
> - What happened to the child after receiving the signal?
> - What did the parent process do in response to losing a child?
> - Did the parent spawn a replacement child? How quickly?
> - What signal did the parent receive when its child died? (Hint: think about which signal the kernel sends automatically to a parent when a child exits)

---

### 📌 Task 3.5 — Check for Zombies

After all the signal activity in this phase, check your system for any zombie processes.

> **Document in your report:**
> - The command you used to check for zombies
> - Did you find any? If yes — the PID, PPID, and how you resolved them
> - If no zombies — explain why your application did not produce them even though you killed child processes

---

# Phase 4 — Priority Tuning
### *"Give your application the right share of CPU"*

> **Estimated Time:** 25 – 30 minutes  
> **Topics:** Nice values · `renice` · CPU scheduling · `ps -eo` · `top` · privilege boundaries

Not all processes should have equal CPU time. In a real production system, critical services run at higher priority than background jobs. In this phase you will experiment with your application's scheduling priority and observe the real effects.

---

### 📌 Task 4.1 — Check the Starting Priority

Find the current nice value of your application and all its child processes.

> **Document in your report:**
> - The current nice value of the main process
> - The nice values of any child processes — are they the same as the parent?
> - What priority does this nice value represent on the scale of `-20` to `+19`?
> - Is this an appropriate priority for this application in a production environment?

---

### 📌 Task 4.2 — Lower the Priority (Be More Polite)

Increase the nice value of your application to `+10` — making it more polite to other processes.

> **Document in your report:**
> - The exact `renice` command you used
> - Did the change apply to child processes automatically or only to the main process?
> - Generate some load on your application (same method as Phase 2). Does it feel slower to respond?
> - Check the CPU distribution in `top` — has the scheduler given your application less CPU time compared to Phase 2?
> - What type of real-world scenario would justify running this application at a lower priority?

---

### 📌 Task 4.3 — Attempt to Raise Priority (Become Greedy)

Try to set your application's nice value to `-5` without elevated privileges, then try again with `sudo`.

> **Document in your report:**
> - What happened without elevated privileges? Paste the exact error.
> - What happened with `sudo`?
> - Explain the security reasoning behind why only root can set negative nice values. Use your application as a concrete example — what would happen if any user could set your web server to `-20`?

---

### 📌 Task 4.4 — Find the Right Priority

Decide what nice value your chosen application should actually run at in a real production system. Set it there.

> **Document in your report:**
> - The nice value you chose and your reasoning
> - Consider: Is this a critical user-facing service or a background task? Does it compete with other important processes? Does it need to respond quickly or can it wait?
> - The exact command you used to set the final priority
> - The final `ps -eo` output confirming the new value

---

# Phase 5 — Job Survival
### *"Make your application outlive the terminal that started it"*

> **Estimated Time:** 25 – 35 minutes  
> **Topics:** `SIGHUP` · `nohup` · `disown` · job control · background jobs · session management · `fg` · `bg`

In real production environments, applications must keep running even when the engineer who started them logs out. In this phase you will test exactly what happens to your application when its controlling terminal disappears — and practice the two main ways to prevent it from dying.

---

### 📌 Task 5.1 — Understand Your Application's Current Session

Before testing survival, understand what terminal session your application currently belongs to.

> **Document in your report:**
> - What TTY or pts does your application show in `ps`?
> - Is it attached to a terminal at all, or does it show `?` in the TTY column?
> - If it shows `?` — explain what this means and why `systemctl`-started services always show this
> - If it shows a real TTY — explain what risks this creates

---

### 📌 Task 5.2 — The Terminal Close Test

Stop your application. Restart it manually from the terminal (not via `systemctl`) so it is attached to your current session. Then close the terminal and check whether it survived.

> For applications started with `systemctl`, you will need to stop the service first and start it manually:

```
# nginx
sudo nginx

# Apache
sudo apache2ctl start
# or
sudo httpd

# Redis
redis-server

# Python HTTP server
python3 -m http.server 8080
```

> **Document in your report:**
> - How you started the application manually
> - Its PID before you closed the terminal
> - Whether it survived after the terminal closed
> - If it died — what signal killed it and what mechanism triggered that signal?

---

### 📌 Task 5.3 — Protect It With nohup

Restart your application using `nohup` so it is immune to `SIGHUP`.

> **Document in your report:**
> - The exact `nohup` command you used
> - The PID assigned
> - The message `nohup` printed — what does it mean?
> - Where is output going now? What is in `nohup.out` after you generate some activity?
> - Close the terminal and verify the application is still running in a new terminal
> - Is it genuinely alive and serving requests, or just a zombie?

---

### 📌 Task 5.4 — Protect It With disown

Restart your application in the background, then `disown` it.

> **Document in your report:**
> - How you started it in the background
> - What `jobs` showed before and after disowning
> - The exact `disown` command
> - Close the terminal and verify it survived in a new terminal
> - What is the practical difference between this method and `nohup`?

---

### 📌 Task 5.5 — When Would You Use Each?

Based on your experiments with your real application, answer these questions for your specific choice:

> **Document in your report:**
> - If you needed to start nginx (or your application) and guarantee it keeps running after you log out from an SSH session — which method would you use and why?
> - If you forgot to use `nohup` or `disown` before starting the process — what are your options?
> - For a real production deployment, neither `nohup` nor `disown` is the actual best practice. What is? Why?

---

# Phase 6 — Deep Investigation
### *"Read the kernel's own records about your application"*

> **Estimated Time:** 25 – 35 minutes  
> **Topics:** /proc filesystem · process introspection · open files · memory maps · limits · environment · practical forensics

The /proc filesystem is the kernel's live notebook about every running process. In this final phase you will read it systematically and build a complete forensic profile of your application — the kind of investigation a sysadmin would do when diagnosing a mysterious production issue.

---

### 📌 Task 6.1 — Full Status Report

Read your application's complete /proc status and explain every field.

> **Document in your report:**
> - The full contents of `/proc/<PID>/status`
> - Explain these fields in your own words using your application's actual values:
>   - `Name`, `State`, `Pid`, `PPid`
>   - `VmPeak`, `VmRSS`, `VmSwap` (what do these memory values tell you?)
>   - `Threads` (how many threads is your application running?)
>   - `SigPnd`, `SigBlk`, `SigIgn`, `SigCgt` (which signals is your application blocking or ignoring?)

---

### 📌 Task 6.2 — What Files Does It Have Open?

List all file descriptors your application currently has open.

> **Document in your report:**
> - The total number of open file descriptors
> - What is file descriptor `0`? File descriptor `1`? File descriptor `2`? (These are standard and the same for every process)
> - Can you identify any application-specific files — config files, log files, socket files, pid files?
> - What does it mean when a file descriptor points to a socket rather than a regular file?

---

### 📌 Task 6.3 — Memory Map

Read your application's memory map from /proc.

> **Document in your report:**
> - The first 20 lines of `/proc/<PID>/maps`
> - Can you identify the application's own executable in the map?
> - Can you identify any shared libraries (`.so` files)?
> - What are the different permission combinations you see (`r`, `w`, `x`, `p`)? What does each mean?

---

### 📌 Task 6.4 — Environment Variables

Read the environment variables your application was started with.

> **Document in your report:**
> - The contents of `/proc/<PID>/environ` (use `tr` to make it readable)
> - Can you find the `PATH` variable? What directories does it include?
> - Are there any application-specific environment variables?
> - Why might reading a process's environment variables be useful in a production incident?

---

### 📌 Task 6.5 — Resource Limits

Read the resource limits the kernel has set for your application.

> **Document in your report:**
> - The full contents of `/proc/<PID>/limits`
> - What is the maximum number of open files allowed? Is this sufficient for a busy web server?
> - What is the maximum stack size?
> - Have any limits been set to "unlimited"? Which ones and why might that be appropriate?

---

### 📌 Task 6.6 — The Final Comparison

Compare your application's /proc profile right now to the baseline you took in Phase 1 Task 1.5.

> **Document in your report:**
> - What changed across all the phases of this project?
> - Did memory usage grow or stay constant?
> - Did the number of open files change?
> - Did the process state ever stay in `R` (running) for extended periods, or was it mostly `S` (sleeping)?
> - What surprised you most about your application's behaviour as a Linux process?

---

## 🎯 Project Goals

This project is designed with clear, concrete outcomes in mind. By the time you finish all six phases, you will have achieved the following:

**1. Bridge theory and reality.**
Course concepts like PIDs, signals, /proc, and nice values only become meaningful when applied to software people actually use. This project forces that connection — every command you run has a visible, real effect on a real program.

**2. Build genuine diagnostic skill.**
You will practice the same investigative workflow a professional sysadmin uses when something goes wrong at 3 AM: identify the process, read its /proc profile, send the right signal, monitor its recovery. These are not academic exercises — they are the actual job.

**3. Understand process lifecycle end-to-end.**
From the moment your application starts to the moment it dies, you will observe every stage: how it forks children, how it responds to signals, how it consumes and releases resources, and how it behaves when its controlling terminal disappears.

**4. Develop production thinking.**
You will not just run commands — you will reason about them. Why does this application run at this priority? What would happen if it couldn't handle `SIGHUP`? Should this process survive a terminal close? Answering these questions turns you from someone who knows Linux commands into someone who thinks in Linux.

**5. Practice honest, precise observation.**
A core professional skill is documenting exactly what you saw — not what you expected to see, not what the documentation says should happen. This project trains that discipline. Real output with a wrong explanation is more valuable than a correct explanation with no evidence.

---

## 💡 Hints

These hints contain no commands. They exist to unstick genuine conceptual confusion — not to shortcut the work.

<details>
<summary>Phase 1 — My application shows multiple processes with the same name. Which one is the main process?</summary>

When an application has a master process and workers, the master is typically the one with the lowest PID — it was the first to start and it forked the others. Look at the PPID column. The process whose PPID points to systemd or a shell is the master. The processes whose PPID points to that master are the workers.

</details>

<details>
<summary>Phase 2 — My application's CPU percentage barely changed under load</summary>

This is normal for I/O-bound applications. If your application spends most of its time waiting for network connections or disk reads rather than doing CPU computation, the CPU percentage stays low even under heavy load. The interesting metric to watch instead is the STATE column — does it stay in S (sleeping, waiting for I/O) even while actively handling requests? That tells you more about its behaviour than CPU percentage alone.

</details>

<details>
<summary>Phase 3 — After sending SIGTERM my application restarted automatically. I expected it to just die.</summary>

Some applications are managed by a process supervisor — systemd, supervisord, or a watchdog process — that automatically restarts them if they exit. If your application came back to life after SIGTERM without you doing anything, check whether systemd is watching it. The application's PID will have changed. This is actually production-correct behaviour and worth documenting. It does not mean something went wrong.

</details>

<details>
<summary>Phase 3 — I cannot find any zombie processes after killing child workers.</summary>

Well-written applications handle SIGCHLD properly. When a child exits, the kernel sends SIGCHLD to the parent. A good parent immediately calls wait() to collect the exit code, which prevents zombies from forming. If your application left no zombies, it means its parent process is doing its job correctly. Document this as a positive observation — it shows your application was written with proper process management in mind.

</details>

<details>
<summary>Phase 5 — My application shows ? in the TTY column even when I start it manually.</summary>

Some applications deliberately detach from their controlling terminal as part of their startup sequence — this is called daemonizing. A well-written daemon forks twice and closes its stdin/stdout/stderr on startup so it has no terminal attachment from the moment it starts. If your application does this, you cannot easily test the terminal-close scenario because it has already protected itself. Document what you observe and explain why the application behaves this way.

</details>

<details>
<summary>Phase 6 — The SigIgn and SigCgt fields in /proc/status are hex numbers. How do I read them?</summary>

Each bit in the hexadecimal number represents one signal. Signal 1 is the least significant bit, signal 2 is the next bit, and so on. You can convert the hex to binary and count which bits are set to find which signals are blocked or caught. As a shortcut, search for an online "signal mask decoder" — there are tools specifically for reading these fields. The more important task is understanding what it means: if SIGTERM appears in SigCgt, your application has a custom handler for graceful shutdown. If SIGHUP appears in SigCgt, it reloads on hangup.

</details>

---

*Linux Process Management — Final Assessment · Real Application Management Project*
