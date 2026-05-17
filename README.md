> [!WARNING]
> Only use this on systems you own or have authorisation to test.

# RootRemover (CVE-2026-31431)
Temporarily removes the root password, uses CVE-2026-31431 to allow architecture-independent escalation to root without knowing the current user's password.

Please use responsibly, only on systems you are authorised to test. Whether this method is supported on your system will vary, but it has worked on every Debian-derivative I've tested.

The PoC this is based on essentially replaced your UID with all 0s in /etc/passwd, effectively giving you root. This approach removes root's password instead, so when you run `su` you get instant root. This keeps your user intact for stability, with the caveat that any user on the machine can elevate until cleanup. Due to not being a binary modification, this should also work on all architectures (Aarch64 and x64 tested).

## Usage
```
python exploit.py [ --shell | --clean | --noclean ]
```
With no arguments, the program runs, makes the changes to `/etc/passwd` and then reverts them.
- `--noclean` prevents changes from being reverted after finishing.
- `--clean` will only clean up from a previous run, not run the exploit.
- `--shell` will not clean up, but will spawn a root shell with `su` after running the exploit.

## What Changes?
<img width="478" height="96" alt="image" src="https://github.com/user-attachments/assets/da39bf6e-4231-4081-a8f7-869bf9f73d4a" />

To

<img width="470" height="102" alt="image" src="https://github.com/user-attachments/assets/cd4dc123-c061-4862-9e72-ce7704457932" />

Removing the 'x' that indicates the password is stored in `/etc/shadow` will be treated as if the user has no password at all, allowing an easy login.

## When To Use This

At the time I wrote there there were only well known PoCs for CopyFail, one that overwrote an SUID executable and one that replaced your user's UID with zeroes. Both of these have reasonably different use-cases if you're running a test against a system. SUID overwrites require shellcode tailored to the system's architecture, and modifying your UID requires you to reauthenticate with your current user's password, which you may not know. RootRemover can often be used if those requirements get in the way.

This is designed for if you get a shell with a nologin user like `www-data`, not a machine where you have a known password or know the architecture well enough to prepare shellcode.

### Advantages of this approach:
- Don't need to know current user password
- Is architecture independent
- Works regardless of your current UID
- Keeps your user's UID intact (stability for other tasks to run on the system under your user, avoids inadvertently writing files as root)
- Does not require writing shellcode to SUID binaries, which could cause issues for programs that need them

### Disadvantages of this approach:
- Root must not use '`*LK*`' or '`*NP*`' in password field (could be fixed with chained writes, I just haven't had time to implement this)
- Elevation may be easier to identify via commandline of `su` (no arguments always means root, the UID patch PoC just looks like switching to your own user in the logs)
- Anyone can log in as root, in the previous version you still need your password.
- Appears to mostly work on Debian-based distros; differences in PAM rules and whether /etc/passwd or /etc/shadow are honoured first affect whether this works on an OS

## Credits

The actual exploit code is from [rootsecdev](https://github.com/rootsecdev/cve_2026_31431)'s repo, I'm mostly changing the bytes that get written for the reasons I outlined above.
