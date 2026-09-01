# Lab 04 — Signals & Process Control

Date: 2026-08-25 | User: mahad | Lab: WSL2 Ubuntu 22.04

> How to use: do each task in the terminal first. When it works, paste your
> REAL output below and answer the questions in your own words.
> Scratch notes go in your notebook — only the finished version lives here.

---

## Task 1 — Signal basics (SIGTERM vs SIGKILL)

```bash
# my commands:
cat > ~/signal-demo.sh << 'EOF'
#!/bin/bash
echo "PID: $$"
trap 'echo "caught SIGTERM, cleaning up..."; exit 0' SIGTERM
trap 'echo "caught SIGINT, cleaning up..."; exit 0' SIGINT
while true; do sleep 1; done
EOF
chmod +x ~/signal-demo.sh

# Test SIGTERM (graceful)
~/signal-demo.sh &
PID=$!
sleep 1
kill -15 $PID
wait $PID
echo "Exit code: $?"

# Test SIGKILL (forced)
~/signal-demo.sh &
PID=$!
sleep 1
kill -9 $PID
wait $PID
echo "Exit code: $?"
```

### 1.1 What exit code means "killed by SIGTERM (signal 15)" and why?
```
# (answer):
Exit code 143 (128 + 15). When a process is terminated by a signal, the shell reports 128 + signal_number.
```

### 1.2 What exit code means "killed by SIGKILL (signal 9)"?
```
# (answer):
Exit code 137 (128 + 9). The shell prints "Killed" and returns 137.
```

### 1.3 What does `kill -0 $PID` do?
```
# (answer):
It checks if a process exists without sending a signal. Returns 0 if the process exists (and you have permission), non-zero otherwise. Useful for testing process liveness.
```

---

## Task 2 — Graceful service script

```bash
# my commands:
cat > /root/labs/svc.sh << 'EOF'
#!/bin/bash
LOG="/var/log/svc.log"
LOCK="/tmp/svc.lock"

echo "started at $(date)" >> "$LOG"
echo $$ > "$LOCK"

cleanup() {
    echo "stopped gracefully at $(date)" >> "$LOG"
    rm -f "$LOCK"
    exit 0
}
trap cleanup SIGTERM SIGINT

while true; do
    echo "heartbeat $(date)" >> "$LOG"
    sleep 2
done
EOF
chmod +x /root/labs/svc.sh

# Run and test graceful stop
sudo /root/labs/svc.sh &
PID=$!
sleep 3
sudo kill -15 $PID
wait $PID
cat /var/log/svc.log
ls -l /tmp/svc.lock
```

### 2.1 What appears in the log after graceful shutdown?
```
# (answer):
Heartbeat lines followed by "stopped gracefully at <timestamp>", and the lockfile is removed.
```

### 2.2 What happens if you `kill -9` instead? Why is this dangerous?
```
# (answer):
The process dies instantly without running the trap. The lockfile remains at /tmp/svc.lock, leaving stale state that could prevent future runs or cause confusion.
```

---

## Task 3 — Zombie process observation

```bash
# my commands:
sleep 100 &
PID=$!
kill -9 $PID
ps -o pid,ppid,stat,comm -p $PID
```

### 3.1 What state does the process show immediately after SIGKILL?
```
# (answer):
It shows as `Z` (zombie/defunct) until the parent reaps it with wait().
```

---

## What I learned (2-3 sentences)

- What surprised me: SIGKILL genuinely cannot be caught or ignored — the kernel destroys the process instantly without giving it a single chance to run cleanup code, and zombies (defunct processes) still hold a PID slot because the parent has not called wait() to reap them.
- What I broke and how I fixed it: When testing the service script with kill -9, the trap handler never ran so the lockfile at /tmp/svc.lock was left behind as stale state; the fix was to always try kill -15 first, wait, then manually remove any stale lockfile before escalating to kill -9.
- One thing I will never do again: I will never use kill -9 as a first resort — it bypasses every cleanup path and risks data corruption, orphaned temp files, and broken locks; I always try SIGTERM first, check if it is still alive, then escalate only if needed.
