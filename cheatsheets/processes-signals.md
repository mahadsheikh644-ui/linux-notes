# Cheatsheet: Processes & Signals

## Signal basics
```
kill -l                 # list signals
kill <pid>              # = kill -15 (SIGTERM) polite
kill -15 <pid>          # SIGTERM: graceful, cleanup runs
kill -9 <pid>           # SIGKILL: forced, no cleanup
kill -0 <pid>           # probe: alive? (exit 0 = yes)
killall -15 nginx       # signal by process name
pkill -9 -f pattern     # signal by command-line pattern
```

## Exit status math
- killed by signal N → exit status = **128 + N** (TERM=143, KILL=137)
- exit 0 = clean; 1 = generic error; 2 = misuse of shell builtin

## Process inspection
```
ps aux                    # all processes, wide format
ps auxf                   # process tree
ps -o pid,ppid,stat,comm -p PID   # specific process, state
top / htop               # live view
pgrep -f pattern          # find PID by pattern
cat /proc/PID/status      # status incl. SigCgt (caught signals)
lsof -p PID               # open files of a process
strace -p PID             # trace syscalls of a running process
```

## Process states (STAT column)
R running | S sleeping | D uninterruptible (I/O) | T stopped | Z zombie

## Zombies
- Child died but parent hasn't reaped it → zombie `Z` (holds a PID slot).
- Parent must `wait()`; killing the parent lets init (PID 1) reap orphans.

## systemd shutdown escalation
`systemctl stop svc` → SIGTERM → wait → SIGKILL (that's what "TimeoutStopSec" controls)