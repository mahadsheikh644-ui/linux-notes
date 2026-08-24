# Lesson 04 — kill -15 vs kill -9 (Signals)

Date: 2026-08-18 | Live demo on WSL2 Ubuntu 22.04

## Theory

A "kill" is really just **sending a signal** to a process. The kernel delivers it;
the process (or the kernel) decides what happens. Signals have numbers and names.

| Signal | Number | Name | Default behavior | Can process catch it? |
|--------|--------|------|------------------|----------------------|
| SIGTERM | 15 | Terminate | Process exits | **YES** — cleanup code runs |
| SIGKILL | 9 | Kill | Process destroyed instantly | **NO** — impossible to catch |

### SIGTERM (-15) — the polite signal
- "Please stop now."
- Process **can** catch it and run cleanup: flush files, close DB connections,
  remove temp files, write final state, tell dependents it's leaving.
- Used by: systemd stopping services, `shutdown`, package managers, Docker stop.
- If the process ignores it, nothing happens (process keeps running).

### SIGKILL (-9) — the sledgehammer
- "Stop NOW. No discussion."
- Kernel terminates the process immediately. **No cleanup, no chance to save state.**
- Risks: data corruption (buffers not flushed), orphaned temp files, corrupted
  locks, dependent processes broken.
- Only to be used when SIGTERM has failed after a reasonable wait.

### Rule of thumb (interview answer)
> Try `kill -15` first. Wait a few seconds (check `ps`/`systemctl status`).
> If the process is still alive, escalate to `kill -9`.

## Live demo evidence (run on 2026-08-18)

A Python process with a SIGTERM handler:

**Part 1 — SIGTERM:** handler ran, printed "cleaning up...", exited with code 0.
```
[SIGTERM received] cleaning up...
[cleanup done] writing final state, exiting gracefully
wait returned: 0
```

**Part 2 — SIGKILL:** no handler ran, no cleanup message, shell reported:
```
4432 Killed  python3 d4_sigdemo.py
wait returned: 137   # 128 + 9 = killed by signal 9
```

### Key facts the demo proved
1. `SigCgt` in `/proc/<pid>/status` shows which signals the process catches
   (bitmask 0x4000 = bit 15 = SIGTERM is handled).
2. `kill -0 <pid>` = "is the process alive?" (does NOT send a signal).
3. Exit status math: `wait` returns **128 + signal number** when killed by a
   signal (137 = 128+9). Exit code 0 = clean exit.
4. `Killed` in shell output = the shell noticed SIGKILL (shells don't print
   anything for SIGTERM deaths, they just exit with 143).

## Useful commands

```bash
kill -l                     # list all signals with numbers
kill -15 <pid>              # polite stop (default: kill <pid> == SIGTERM)
kill -9 <pid>               # force kill
kill -0 <pid>               # check existence (0 = alive)
killall -15 nginx           # signal by name
pkill -9 -f "myscript.py"   # signal by pattern
ps -o pid,stat,comm -p <pid># process state (S = sleeping, Z = zombie, R = running)
cat /proc/<pid>/status | grep SigCgt   # which signals are caught
```

## Interview tip
Answer in one breath: *"SIGTERM is the graceful shutdown that lets the app clean
up; SIGKILL is the last resort that can't be intercepted and risks data loss.
I always try TERM first, wait, check if it's still running, then escalate."*

## To practice
See labs/04-signals-lab.md