# Lab 04 — Homework: Signals & Process Control

Goal: reproduce the demo **from memory**, then extend it.

## Task 1 — Reproduce (no copying from the lesson)
1. Write a small script (bash or python) that:
   - prints its PID
   - sleeps/loops forever
2. Run it in the background, note the PID.
3. Send `kill -15` — check with `kill -0` whether it died.
4. Start another, send `kill -9` — check the shell message and `echo $?`.

**Expected:** SIGTERM exit → `$?` of `wait` is 143 (128+15). SIGKILL → 137 (128+9)
and the shell prints `Killed`.

## Task 2 — The graceful service (the sysadmin skill)
Write a bash script `/root/labs/svc.sh` that:
- On start: writes "started at <date>" to `/var/log/svc.log`, then loops forever,
  appending a heartbeat line every 2s.
- Traps SIGTERM: writes "stopped gracefully at <date>", removes a temp lockfile,
  then exits 0.
- Traps SIGINT (Ctrl+C) the same way.
- Creates `/tmp/svc.lock` at start, deletes it on graceful stop.

Run it, then kill it gracefully. Verify:
- `cat /var/log/svc.log` shows heartbeat + "stopped gracefully"
- `/tmp/svc.lock` is gone

Then run it again and `kill -9` it. Verify the lockfile is **still there**
(proof of why SIGKILL is dangerous — leftovers).

## Task 3 — Zombie observation (bonus, shows process states)
```bash
sleep 100 &
PID=$!
kill -9 $PID
ps -o pid,ppid,stat,comm -p $PID   # likely shows defunct/zombie until reaped
```

## Task 4 — Check answers (test yourself)
1. Which signal can a process NEVER ignore?
2. What exit code means "killed by signal 9"?
3. Name one real scenario where SIGTERM-first matters.
4. What does `kill -0` do?

## Done when
You can explain, out loud, the difference between SIGTERM and SIGKILL using the
exit-code math (128+n) and the cleanup story.