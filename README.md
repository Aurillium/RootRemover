# RootRemover (CVE-2026-31431)
Temporarily removes the root password, uses CVE-2026-31431 to allow escalation to root without knowing the current user's password.

Please use responsibly, only on systems you are authorised to test

Advantages:
- Don’t need to know current user password
- Works on any UID (existing PoC could be modified to work on less than 4 digit too and could chain for more digits)
- Keeps your user’s UID intact (for other tasks running on the system etc)

Disadvantages:
- Root must not have hash in /etc/passwd (this is a security concern in itself)
- Root must not use ‘*LK*’ or ‘*NP*’ in password field (could be fixed with chained writes)
- Elevation may be easier to identify via commandline of ‘su’ (no arguments always means root, UID patch PoC just looks like switching to your own user in the logs. This is a flaw shared with the first PoC at https://copy.fail)
