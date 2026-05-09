# RootRemover (CVE-2026-31431)
Temporarily removes the root password, uses CVE-2026-31431 to allow escalation to root without knowing the current user's password.

Please use responsibly, only on systems you are authorised to test. Currently only seems to work on Debian derivatives.

Actual exploit code is from [rootsecdev](https://github.com/rootsecdev/cve_2026_31431)'s repo, I just change the bytes written for reasons listed below.

The PoC this is based on essentially replaced your UID with all 0s in /etc/passwd, effectively giving you root. This approach removes root's password instead, so when you run `su` you get instant root. This keeps your user intact for stability and grants you root's GID too, with the caveat that any user on the machine can elevate until cleanup.

Advantages of this approach:
- Don’t need to know current user password
- Works on any UID (existing PoC could be modified to work on less than 4 digit too and could chain for more digits)
- Keeps your user’s UID intact (stability for other tasks to run on the system under your user etc, don't want to start writing files with UID 0)

Disadvantages of this approach:
- Root must not have hash in /etc/passwd (this is a security concern in itself and means that if you can't use this script you may still be able to hash crack)
- Root must not use '`*LK*`' or '`*NP*`' in password field (could be fixed with chained writes, I just haven't yet)
- Elevation may be easier to identify via commandline of `su` (no arguments always means root, UID patch PoC just looks like switching to your own user in the logs because that's what's happening)
- Anyone can log in as root, in the previous version you still need your password. This is designed more for if you get a shell with something like `www-data`, not a machine where you have a known password.
- Appears to mostly work on Debian-based OSes so far due to differences in whether /etc/passwd or /etc/shadow are honoured first (possibly fixable but likely time consuming)

In my opinion the advantages of this approach outweigh the disadvantages, so I decided to write it.
