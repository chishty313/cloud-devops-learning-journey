# Module 01 — Linux & Shell for DevOps

> *You don't have to know all of Linux. You have to know the 20% of Linux that shows up in 80% of on-call incidents and interview whiteboards.*

## The story

Every DevOps interview eventually pulls up a terminal on a Linux box and says: *"a process is eating all the CPU, tell me how to find and kill it"*, or *"this service isn't reachable from the outside, walk me through your debugging"*, or *"write me a bash script that rotates logs older than seven days"*. If you fumble the terminal, no amount of Terraform vocabulary will save you. Linux fluency is the base of the pyramid.

The good news is you already know Linux at the level of *"I've used a terminal on my Mac and I've SSH'd into a server once."* This module levels you up to *"I can navigate any Linux box under interview pressure."*

The organising idea to hold onto:

> **Linux is a system of files and processes. Everything you do at the terminal is: read/write a file, or start/stop a process, or connect them together with pipes.**

That's the entire game. Once you accept that "listing users", "starting a web server", "watching network traffic", "checking disk space", and "reading logs" are all just *files or processes* dressed up in different commands, the tools stop feeling arbitrary.

The module covers six concepts, in this order:

1. **The filesystem hierarchy that matters for DevOps** — `/etc`, `/var/log`, `/proc`, `/sys`, and why they exist.
2. **Permissions and ownership** — the reason your container "just doesn't have permission" errors happen and how to reason about them.
3. **Processes and services** — `ps`, `top`/`htop`, `systemctl`, `journalctl`; how a modern Linux system runs and supervises daemons.
4. **Networking from the terminal** — `ss`, `curl`, `dig`, `nc`, `tcpdump`, `traceroute`. What each shows, and the sequence you use them in when "the service is down".
5. **The pipeline** — `grep`, `sed`, `awk`, `find`, `xargs`, `cut`, `sort`, `uniq`. The DevOps interview *loves* one-liners that answer questions like "which IP hit us the most in the last hour".
6. **Bash scripting that doesn't fall over** — `set -euo pipefail`, `trap`, functions and exit codes, positional args, the difference between `[[` and `[`.

We'll do each concept as: a *why-it-exists* paragraph, then a *hands-on you run*, then a *lesson learned* line. The hands-on log for the whole module lives in [`hands-on.md`](./hands-on.md).

---

## Concept 1 — The filesystem hierarchy that matters for DevOps

Historical crash-course in one paragraph: Linux inherits its layout from Unix in the 1970s. Directories are grouped by *what kind of thing lives there*, not *which app owns it*. That's the opposite of Windows/macOS where each app owns a folder in `Program Files` or `/Applications`. On Linux, one app's files are scattered across many directories — binaries in `/usr/bin`, config in `/etc/<app>`, logs in `/var/log/<app>`, runtime data in `/var/lib/<app>`. Once you know the categories, you know where to look on *any* Linux server without asking.

The directories a DevOps engineer opens 100 times a day:

| Directory | What lives there | Why you care |
|---|---|---|
| `/etc` | System-wide configuration files. `/etc/hosts`, `/etc/resolv.conf`, `/etc/ssh/sshd_config`, `/etc/nginx/nginx.conf` etc. | When a service misbehaves after "someone changed something", 90% of the time the change is here. |
| `/var/log` | Log files. `/var/log/syslog`, `/var/log/auth.log`, `/var/log/nginx/access.log`. | First place you tail when something is broken. |
| `/var/lib` | Persistent state (databases, caches, package manager state). `/var/lib/docker` for images/containers, `/var/lib/postgresql` for data. | When "the disk is full" the culprit is usually here. |
| `/proc` | A **virtual** filesystem. Every running process has a directory `/proc/<pid>/`; the kernel exposes runtime info as pseudo-files. `/proc/meminfo`, `/proc/cpuinfo`, `/proc/net/tcp`. | Almost every "show me system state" tool (`ps`, `top`, `free`, `ss`) is really just reading files under `/proc`. Knowing this demystifies half of Linux. |
| `/sys` | Another virtual FS, exposing devices and kernel objects. | You'll rarely edit here, but when a device won't come up it's where you check state. |
| `/tmp` | Temporary. Wiped on boot on most systems (or by systemd-tmpfiles). | Never store anything you need to survive a reboot here. Also — containers *often* have a tiny `/tmp`; blowing it up is a common CI failure. |
| `/root` | Root's home directory. | If you're logged in as root, this is `~`. |
| `/home/<user>` | Each non-root user's home. | Same. |
| `/usr/bin`, `/usr/local/bin`, `/opt` | Installed programs. `/usr/bin` is from the OS package manager; `/usr/local/bin` is admin-installed; `/opt` is often for hand-installed 3rd party. | When "command not found" hits, `which <cmd>` tells you where the binary lives (or doesn't). |
| `/run` | Runtime state (pids, sockets) that shouldn't survive reboot. | Systemd stores unit socket files here. |

The trick is that `/proc` and `/sys` aren't real files on disk — they're doors into the kernel. When you `cat /proc/meminfo`, you're not reading a file, you're asking the kernel a question and the kernel answers by *pretending* to be a file. Once you get this, the "tools" like `top`, `ps`, `free` stop feeling magical — they're just pretty printers over `/proc`.

### Hands-on 1: prove the mental model on your own machine

Your Mac isn't Linux, but Docker Desktop runs Linux VMs, and we'll use one so the exploration is real Linux (not macOS's BSD variant, which behaves differently in subtle ways).

Run this in your terminal:

```bash
docker run --rm -it ubuntu:24.04 bash
```

**Breakdown:**
- `docker run` — start a new container from an image.
- `--rm` — delete the container as soon as we exit. Keeps your machine clean.
- `-it` — interactive, allocate a TTY. Without these two together, you can't actually type into the container.
- `ubuntu:24.04` — image name and tag. `ubuntu` is Canonical's official image; `24.04` is the current LTS.
- `bash` — the command to run inside the container. Overrides the default (which is also bash for this image, but being explicit is a habit worth building).

You should land in a `root@<hash>:/#` prompt. That's a real Ubuntu shell running in a container on your Mac.

Now, inside the container, run these one at a time and observe:

```bash
ls /
ls /etc | head -20
cat /etc/os-release
cat /proc/meminfo | head -5
cat /proc/cpuinfo | grep '^model name' | head -1
ls /var/log
```

**Breakdown of what each reveals:**
- `ls /` — the top of the filesystem tree. You should see the directories from the table above (plus a few Docker-specific ones).
- `ls /etc | head -20` — a peek at how much lives under `/etc` on a *bare* Ubuntu image (this container has almost no services installed, so the list is short — production servers have hundreds).
- `cat /etc/os-release` — the machine-readable identity of the OS. Every distro puts something here. Scripts that need to behave differently on Ubuntu vs. RHEL vs. Alpine look at this file.
- `cat /proc/meminfo | head -5` — the *live* memory state of the container, being read from the kernel via `/proc`. That's the raw data `free -h` prettifies.
- `cat /proc/cpuinfo | grep '^model name' | head -1` — same trick for the CPU. Notice: this container has *no* CPU of its own; you're seeing the host's CPU because containers share the host kernel.
- `ls /var/log` — see what logs a bare Ubuntu image ships with (spoiler: almost none, because no services are running — logs appear when services do).

**When you're done, type `exit`** to leave the container. Because of `--rm`, it vanishes.

Paste back:
1. The output of `cat /etc/os-release`.
2. The first line from `cat /proc/cpuinfo | grep '^model name'` (I want to confirm your Docker Desktop is exposing the host CPU as expected).
3. Anything under `ls /var/log` that surprises you (or "empty/small" if it isn't).

Once we've seen your outputs, I'll write your terminal session into [`hands-on.md`](./hands-on.md) and we move to Concept 2 (permissions and ownership).
