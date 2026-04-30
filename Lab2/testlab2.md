# 🧪 Lab 02 — Listing & Finding Processes

### 🎯 Topic

Listing / Finding Processes

### 📊 Level

Beginner → Intermediate

### ⏱ Duration

40 – 55 minutes

---

## 🎬 Mission Brief

A critical alert hits **NovaByte Technologies** — the system is slow.

Your lead says:

> “Something’s eating the CPU. Find it.”

You have terminal access and limited time.
Your job: **identify the process and understand it completely.**

---

## 📘 What You Will Learn

* Use `ps`, `pgrep`, `pidof`, `top`
* Understand `ps` output columns
* Explore `/proc` for deep inspection
* Filter processes by CPU, memory, user
* Interpret process states (`R`, `S`, `D`, `Z`, `T`)

---

# 🔹 Task 2.1 — Full Process Snapshot

```bash
ps aux
```

**Sample output:**

```
USER   PID %CPU %MEM VSZ   RSS TTY STAT START TIME COMMAND
root     1  0.0  0.1 ...
student 2988 0.0 0.0 ps aux
```

💡

* `%CPU` → CPU usage
* `%MEM` → RAM usage
* `VSZ` → virtual memory
* `RSS` → actual RAM
* `STAT` → process state

<p align="center">
  <img src="https://raw.githubusercontent.com/harigoviind/processmnggmnt-project-/c1c5837d92d94d02a4d419b9580f566c32afbd18/Lab2/Screenshot%202026-04-30%20124948.png" width="500">
</p>

---

## 🔢 Count Processes

```bash
ps aux | wc -l
```

<p align="center">
  <img src="https://raw.githubusercontent.com/harigoviind/processmnggmnt-project-/c1c5837d92d94d02a4d419b9580f566c32afbd18/Lab2/Screenshot%202026-04-30%20125148.png" width="450">
</p>

---

# 🔹 Task 2.2 — Find a Process (2 Ways)

```bash
sleep 999 &
```

### Method 1 — `pgrep`

```bash
pgrep sleep
```

<p align="center">
  <img src="https://raw.githubusercontent.com/harigoviind/processmnggmnt-project-/c1c5837d92d94d02a4d419b9580f566c32afbd18/Lab2/Screenshot%202026-04-30%20125306.png" width="450">
</p>

---

### Method 2 — `ps + grep`

```bash
ps aux | grep sleep
```

<p align="center">
  <img src="https://raw.githubusercontent.com/harigoviind/processmnggmnt-project-/c1c5837d92d94d02a4d419b9580f566c32afbd18/Lab2/Screenshot%202026-04-30%20125343.png" width="500">
</p>

💡

* Shows **extra line (grep itself)**
* `pgrep` is cleaner for scripts

---

# 🔹 Task 2.3 — Advanced `pgrep`

```bash
pgrep -a sleep
```

<p align="center">
  <img src="https://raw.githubusercontent.com/harigoviind/processmnggmnt-project-/c1c5837d92d94d02a4d419b9580f566c32afbd18/Lab2/Screenshot%202026-04-30%20125410.png" width="450">
</p>

```bash
pgrep -u $USER
```

<p align="center">
  <img src="https://raw.githubusercontent.com/harigoviind/processmnggmnt-project-/c1c5837d92d94d02a4d419b9580f566c32afbd18/Lab2/Screenshot%202026-04-30%20125435.png" width="450">
</p>

---

# 🔹 Task 2.4 — Process Tree

```bash
ps auxf
```

<p align="center">
  <img src="https://raw.githubusercontent.com/harigoviind/processmnggmnt-project-/c1c5837d92d94d02a4d419b9580f566c32afbd18/Lab2/Screenshot%202026-04-30%20125505.png" width="500">
</p>

💡

* `\_` → parent-child relationship
* Helps visualize process hierarchy

---

# 🔹 Task 2.5 — Live Monitoring (`top`)

```bash
top
```

<p align="center">
  <img src="https://raw.githubusercontent.com/harigoviind/processmnggmnt-project-/c1c5837d92d94d02a4d419b9580f566c32afbd18/Lab2/Screenshot%202026-04-30%20125534.png" width="550">
</p>

### Useful keys:

* `M` → sort by memory
* `P` → sort by CPU
* `u` → filter by user
* `1` → per-core view
* `q` → quit

---

# 🔹 Task 2.6 — Inspect `/proc`

```bash
cat /proc/<PID>/status
```

<p align="center">
  <img src="https://raw.githubusercontent.com/harigoviind/processmnggmnt-project-/c1c5837d92d94d02a4d419b9580f566c32afbd18/Lab2/Screenshot%202026-04-30%20125622.png" width="500">
</p>

---

```bash
cat /proc/<PID>/cmdline | tr '\0' ' '
```

<p align="center">
  <img src="https://raw.githubusercontent.com/harigoviind/processmnggmnt-project-/c1c5837d92d94d02a4d419b9580f566c32afbd18/Lab2/Screenshot%202026-04-30%20125806.png" width="500">
</p>

💡

* `/proc` gives **kernel-level truth**
* More reliable than `ps` formatting

---

# 🧠 Reflection Questions

1. Why does `ps | grep` show extra lines?
2. What does `S` state mean? Compare with `R`, `D`
3. Why can VSZ be large without issues?
4. Why is `/proc/<PID>/cmdline` more accurate?
5. How would you safely kill user-owned processes only?

---
