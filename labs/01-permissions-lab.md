# Lab 01 — Homework: Permissions Mastery

Goal: reproduce the demo **from memory**, then solve real scenarios.
Rules: do it yourself first. Only check the lesson if stuck >10 min.
At the end: fill in the "What I learned" section and commit to GitHub.

## Task 1 — SUID/SGID/sticky (from memory)
1. Create a directory `/tmp/perm-lab` (use sudo).
2. Create a file `/tmp/perm-lab/tool` owned by root.
3. Make it SUID. Verify the `s` appears in `ls -l`.
4. Create a shared directory `/srv/team` owned by group `devops` (create the group first).
5. Set SGID on it. Create a file as a different user inside — check the group.
6. Set sticky on `/tmp/perm-lab`. Try to delete a file owned by another user — what happens?

## Task 2 — umask
1. In a fresh shell, set `umask 077`, touch a file, check its perms.
2. Set `umask 002`, repeat. Explain the difference in 2 lines.
3. Where would you set a *permanent* umask for one user? Where for all users?

## Task 3 — ACLs
1. Create `/srv/app/config.yaml` owned by root:devops with 640.
2. Use `setfacl` to give user `alice` read access (create alice first).
3. Verify with `getfacl` and confirm the `+` in `ls -l`.
4. Now try the same thing with classic permissions — what's the problem with
   adding alice to the group? (one-line answer)

## Task 4 — Permission reading drill
Write the numeric mode for each of these (no calculator):
- `-rwxr-xr--`
- `-rwsr-xr-x`
- `drwxrwsr-x`
- `drwxrwxrwt`
Then answer: what is the difference between `chmod 1777` and `chmod 3777` on a directory?

## Task 5 — The sysadmin scenario (this is the interview question)
You're on call. A user says: "I can't write to /var/www/html anymore.
It worked last week." The directory is owned by root:www-data, perms 755.
- What are the possible causes? List at least 3.
- What commands do you run to diagnose each one?
- What is the correct fix (and why NOT `chmod 777`)?

## Expected outcomes
- SUID `s`, SGID `s`, sticky `t` visible in ls output
- umask understood and demonstrated
- ACL `+` visible, getfacl output understood
- Task 5 answered in writing — this goes in your notes and interview prep
