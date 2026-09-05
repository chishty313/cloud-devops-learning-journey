# Module 01 — Linux & Shell for DevOps

> *You don't need to know all of Linux. You need to know the 20% of Linux that shows up in 80% of on-call incidents and interview whiteboards.*

## The story

Every Cloud/DevOps interview eventually pulls up a terminal on a Linux box and says something like:

- *"A process is eating all the CPU. Find it and kill it."*
- *"This service isn't reachable from the outside. Walk me through debugging."*
- *"Write a bash script that rotates any log older than seven days."*

If the terminal is unfamiliar under pressure, no amount of Terraform vocabulary saves you. Linux fluency is the base of the pyramid every other Cloud/DevOps skill sits on.

The good news: you already know Linux at the level of *"I've used a terminal, I've SSH'd into a server once."* This module levels you up to *"I can navigate any Linux box under pressure."*

## The single mental model that unlocks Linux

Hold this in your head — everything else follows:

```mermaid
flowchart LR
    subgraph linux["What Linux is, at its core"]
        Files[("📄  Files<br/>(everything, even<br/>hardware, is a file)")]
        Procs["⚙️  Processes<br/>(running programs)"]
        Pipes(["🔗  Pipes<br/>(files ↔ processes ↔ each other)"])
    end
    Files <-.-> Pipes <-.-> Procs
```

> **Linux is a system of files and processes, connected by pipes. Every command you'll ever run is one of: read a file, write a file, start a process, stop a process, or connect them together.**

That's the whole game. Once you accept that "listing users", "starting a web server", "watching network traffic", and "checking disk space" are all just *files or processes* dressed up in different commands, the tools stop feeling arbitrary.

---

## What this module covers, in order

```mermaid
flowchart LR
    C1["1 · Filesystem<br/>hierarchy"] --> C2["2 · Permissions<br/>& ownership"]
    C2 --> C3["3 · Processes<br/>& services"]
    C3 --> C4["4 · Networking<br/>from the CLI"]
    C4 --> C5["5 · The Unix<br/>pipeline"]
    C5 --> C6["6 · Bash<br/>scripting"]
```

Each concept follows a fixed shape: *why-it-exists* → *hands-on you run* → *lesson learned*. The hands-on log for the whole module lives in [`hands-on.md`](./hands-on.md).

---

## Concept 1 — The filesystem hierarchy that matters

### Historical context in one paragraph

Linux inherited its layout from Unix in the 1970s. Directories are grouped by *what kind of thing lives there*, not *which app owns it*. That's the opposite of Windows/macOS, where each app owns a folder in `Program Files` or `/Applications`. On Linux, one app's files are scattered across many directories — binaries in `/usr/bin`, config in `/etc/<app>`, logs in `/var/log/<app>`, runtime data in `/var/lib/<app>`. Once you know the categories, you know where to look on *any* Linux server without asking.

### The directories a DevOps engineer opens every day

```
/                       ← the root of the whole tree
├── etc/                ← system configuration
├── var/
│   ├── log/            ← log files
│   └── lib/            ← persistent state (databases, package DBs)
├── proc/               ← virtual: live view of the running kernel
├── sys/                ← virtual: kernel objects (devices, drivers)
├── tmp/                ← temporary, often wiped on reboot
├── run/                ← runtime state (pids, sockets)
├── home/<user>/        ← users' home directories
├── root/               ← root user's home
├── usr/
│   ├── bin/            ← programs from the OS package manager
│   └── local/bin/      ← programs installed by the admin
├── opt/                ← optional / hand-installed 3rd-party
├── dev/                ← device files (disks, terminals)
├── boot/               ← the kernel and bootloader files
└── mnt/, media/, srv/  ← mount points and misc conventions
```

The ones that dominate your day-to-day, and why they matter:

| Directory                            | What lives there                                                                                                        | Why you care                                                                                                                                        |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`/etc`**                           | System-wide configuration files: `/etc/hosts`, `/etc/resolv.conf`, `/etc/ssh/sshd_config`, `/etc/nginx/nginx.conf`, ... | When a service misbehaves after *"someone changed something"*, 90% of the time the change is here.                                                  |
| **`/var/log`**                       | Log files: `/var/log/syslog`, `/var/log/auth.log`, `/var/log/nginx/access.log`.                                         | The first place you `tail -f` when something breaks.                                                                                                |
| **`/var/lib`**                       | Persistent state — databases, caches, package-manager state. `/var/lib/docker` for images/containers.                   | When *"the disk is full"*, the culprit almost always lives here.                                                                                    |
| **`/proc`**                          | A **virtual** filesystem — every running process has a directory `/proc/<pid>/`; the kernel exposes runtime info as pseudo-files. `/proc/meminfo`, `/proc/cpuinfo`, `/proc/net/tcp`. | Almost every *"show me system state"* tool (`ps`, `top`, `free`, `ss`) is really just reading files under `/proc`. Understanding this demystifies half of Linux. |
| **`/sys`**                           | Another virtual FS, exposing devices and kernel objects.                                                                | Rarely edit; check state when a device won't come up.                                                                                               |
| **`/tmp`**                           | Temporary. Wiped on boot on most systems (or by systemd-tmpfiles).                                                      | Never put anything you need to survive a reboot here. In containers, `/tmp` is often tiny — a common CI failure.                                    |
| **`/root`**                          | Root's home directory.                                                                                                  | When you're logged in as root, this is `~`.                                                                                                         |
| **`/home/<user>`**                   | Each non-root user's home.                                                                                              | Same as above for regular users.                                                                                                                    |
| **`/usr/bin`, `/usr/local/bin`, `/opt`** | Installed programs.                                                                                                     | When *"command not found"* strikes, `which <cmd>` tells you where the binary lives (or doesn't).                                                    |
| **`/run`**                           | Runtime state (pids, sockets) that shouldn't survive reboot.                                                            | Systemd stores unit socket files here.                                                                                                              |

### The `/proc` insight that changes how you think about Linux

`/proc` and `/sys` aren't real files on disk. They're **doors into the kernel**.

```mermaid
flowchart LR
    User["you: cat /proc/meminfo"]
    Kernel["Linux kernel"]
    Memory[("real memory<br/>subsystem")]

    User -->|"read()"| Kernel
    Kernel -->|"fabricates a<br/>plain-text answer"| User
    Kernel -.->|"queries live state"| Memory
```

When you `cat /proc/meminfo`, you're **not** reading a file — you're asking the kernel a question, and the kernel is answering by *pretending* to be a file. That trick (making everything look like a file) is why the same tools — `cat`, `grep`, `cut` — work everywhere on Linux.

Once you get this, tools like `top`, `ps`, `free`, `df`, `ss` stop feeling magical. They're pretty-printers over `/proc` and `/sys`.

### Hands-on 1: prove the mental model in a real Linux container

Your Mac is Unix-flavoured but not Linux, and macOS's BSD `ls`, `sed`, and `grep` differ from GNU versions in ways that will bite you later. Every hands-on in this module runs inside a real Linux container — spin one up with Docker Desktop:

```bash
docker run --rm -it ubuntu:24.04 bash
```

**Command breakdown:**
- `docker run` — start a new container from an image.
- `--rm` — delete the container the moment we `exit`. Keeps the machine clean.
- `-it` — **`-i`** keeps stdin open, **`-t`** allocates a TTY. Together they mean "give me an interactive shell." Without them you can't type into the container.
- `ubuntu:24.04` — the image's name and tag. `ubuntu` is Canonical's official image; `24.04` is the current LTS.
- `bash` — the command to run inside the container. Overrides the image's default (which is also bash for this image, but being explicit is a habit worth building).

You should land in a `root@<hash>:/#` prompt — a real Ubuntu shell running in a container on your machine.

Inside the container, run each of these and observe:

```bash
ls /                                                   # top of the tree
ls /etc | head -20                                     # some system config
cat /etc/os-release                                    # machine-readable OS identity
cat /proc/meminfo | head -5                            # live memory state via /proc
cat /proc/cpuinfo | head -20                           # CPU info via /proc
ls /var/log                                            # bare-image log dir
exit
```

**What each command reveals:**

| Command                                | What you learn                                                                                                                            |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `ls /`                                 | The canonical top-level directories from the tree above.                                                                                  |
| `ls /etc \| head -20`                  | A peek at how much lives under `/etc` on a *bare* Ubuntu image (short — production servers have hundreds of entries).                     |
| `cat /etc/os-release`                  | Machine-readable OS identity. Every distro ships this. Portable scripts read this file to behave differently on Ubuntu vs. RHEL vs. Alpine. |
| `cat /proc/meminfo \| head -5`         | The **live** memory state of the container, being read from the kernel via `/proc`. This is the raw data `free -h` prettifies.            |
| `cat /proc/cpuinfo \| head -20`        | Same trick for the CPU. Notice: the container has no CPU of its own — it sees the host's CPU because containers share the host kernel.    |
| `ls /var/log`                          | See what logs a bare Ubuntu image ships with. Spoiler: very few — most logs appear once services start running.                           |

> [!IMPORTANT]
> **Architecture gotcha (Apple Silicon / ARM readers).**  On x86 Linux, `/proc/cpuinfo` includes a `model name : ...` line. On ARM64 Linux (which is what Docker Desktop runs on Apple Silicon Macs), there is **no `model name` field** — the ARM kernel uses different field names (`CPU implementer`, `CPU part`, `CPU architecture`).
>
> This is why a naive `grep '^model name' /proc/cpuinfo` command from a tutorial written for x86 will silently return nothing on ARM. It's a small example of a very large lesson: **CLI tools that assume a specific format across Linuxes will silently fail across architectures.** Cross-architecture awareness matters as soon as you touch containers, cloud instances (AWS Graviton, Azure Ampere), and multi-arch images.

### What to look for in your outputs

- `/etc/os-release` should tell you it's Ubuntu 24.04 (Noble Numbat).
- `/proc/meminfo` shows total, free, and available memory *for the container's cgroup*, not the host — this is the same view your app inside a container sees.
- `/var/log` on a bare Ubuntu image is small (`alternatives.log`, `apt`, `bootstrap.log`, `btmp`, `dpkg.log`, `faillog`, `lastlog`, `wtmp`). Two useful ones to remember:
  - `btmp` — failed login attempts (binary; view with `lastb`).
  - `wtmp` — successful logins/logouts (binary; view with `last`).
  - `lastlog` — last login time per user (binary; view with `lastlog`).

### Concept 1 — takeaway

- 🧭 Linux directory conventions are **shared across every distro** — once you learn them, you can land on any Linux box and know where to look.
- 🔍 **`/proc` is the door to the kernel.** Every "system state" tool reads from it. When a tool doesn't exist or misbehaves, you can often get the raw answer with `cat /proc/<something>`.
- ⚠️ **Format assumptions across architectures will silently fail.** Design commands defensively: check for empty output, or look for stable substrings (e.g. `grep -iE 'model|processor'` catches both x86 and ARM).

---

## Concept 2 — Permissions & ownership

### The three questions behind every "Permission denied"

Every `Permission denied` you'll ever see comes down to answering three questions, in order:

```mermaid
flowchart LR
    Q1["1. Who am I?<br/>(what identity does<br/>my process have?)"] --> Q2["2. Who owns the file,<br/>and what group is it in?"] --> Q3["3. What do the file's<br/>permission bits say<br/>each of those can do?"] --> V{{"Allow<br/>or deny?"}}
```

The Linux permission system is startlingly small — a couple of dozen bytes per file — but it's the foundation the entire Unix security model is built on, and the exact same mental model shows up in container security, Kubernetes `securityContext`, and cloud IAM analogies. Learning it well pays interest forever.

### The 9-bit permission model

Every file and directory carries three sets of three bits: what the **owner** can do, what the **group** can do, and what **other** (everyone else) can do.

```
     ┌────── owner ──────┐  ┌────── group ──────┐  ┌────── other ──────┐
       r        w        x    r        w        x    r        w        x
       │        │        │
      read    write   execute       (repeated for group and other)
```

`ls -l` displays these as a 10-character string:

```
    -rwxr-xr--   ← 1 file-type char + 9 permission bits
    │└─┬─┘└─┬─┘└─┬─┘
    │  owner group other
    │
    └── file type: - regular file
                    d directory
                    l symlink
                    c character device (terminals, ttys)
                    b block device (disks)
                    s socket
                    p named pipe (FIFO)
```

Written as **octal** (the "chmod 755" you've seen everywhere), each triad is 3 bits = 0–7:

| Value | Bits  | Meaning       |
| ----- | ----- | ------------- |
| 7     | `rwx` | read + write + execute |
| 6     | `rw-` | read + write |
| 5     | `r-x` | read + execute |
| 4     | `r--` | read only |
| 3     | `-wx` | write + execute (rare) |
| 2     | `-w-` | write only (rarer) |
| 1     | `--x` | execute only |
| 0     | `---` | nothing |

So `chmod 755 foo` means: owner `rwx`, group `r-x`, other `r-x`. That's the default for an executable file you want everyone to be able to run.

### The gotcha: what `x` means on a **directory**

- On a **file**, `x` means "this can be executed as a program."
- On a **directory**, `x` means "you may `cd` into it and access files inside it by name."

**Without `x` on a directory, you cannot reach any file within it — even if the file's own bits would allow you.** That's why directories default to `755` even when the files inside are just data — the `x` on the dir is what makes the files reachable.

Think of a directory's `x` as "the key to the door"; `r` on a directory only lets you *list* what's inside (see the names).

### `chmod` in symbolic form — often clearer than octal

Instead of memorising octal, you can add or remove specific bits:

```bash
chmod u+x script.sh     # give the owner (u) execute
chmod g-w file          # remove write from group (g)
chmod o=r file          # set 'other' (o) to read-only exactly
chmod a+r file          # everyone (a=all) gets read
chmod u=rwx,g=rx,o=     # set all three at once
```

Both forms produce the same file mode — use whichever reads clearer for the change you're making.

### Owner, group, and root

- **Owner** and **group** are named identities that live in `/etc/passwd` and `/etc/group`.
- **Root** (user id 0) **bypasses the 9-bit check entirely**. Root can read and write anything on the filesystem, ignoring permission bits.

> [!WARNING]
> That last point is why **containers running as root are dangerous.** If a container process gets compromised, the attacker starts with the ability to touch anything the container can see — including mounted volumes from the host. Best practice: run containers as a non-root user (via the `USER` directive in a Dockerfile). The same idea shows up in Kubernetes as `securityContext.runAsNonRoot: true` and `runAsUser: <nonzero-uid>`.

### `umask` — the default permissions for new files

When you `touch` a new file, it doesn't get `777`. It gets a default determined by the **umask**, which is a mask of bits to *strip* from `666` (files) or `777` (directories):

```
umask 022  →  new files get 644, new dirs get 755   ← common default
umask 077  →  new files get 600, new dirs get 700   ← private-by-default
umask 002  →  new files get 664, new dirs get 775   ← "team writable" setups
```

Show your current umask: `umask`. Change it for the current shell: `umask 077`. Make it permanent in `~/.bashrc` or `/etc/profile`. This matters for security-conscious systems where new files must default to private.

### Ownership: `chown` and `chgrp`

Only root can change file ownership:

```bash
chown alice file                  # transfer file to user alice
chown alice:developers file       # user alice, group developers
chgrp developers file             # only change group
chown -R alice:alice /home/alice  # recursive — the "R" is essential
```

> [!TIP]
> `chown -R` is one of the most common recovery commands after a **container volume mount** — when a volume mounts as root and the app can't write to it, you `chown -R app:app /path` to fix it. Every DevOps engineer has typed this at 2am at least once.

### The special bits: setuid, setgid, sticky (brief)

You'll see these in the wild; just recognise them for now.

- **setuid** — `s` where `x` for owner would be. The program runs as its *owner* instead of the caller. This is how `sudo` and `passwd` can perform privileged operations for unprivileged users. Look for `-rwsr-xr-x` on `/usr/bin/passwd`.
- **setgid** — `s` where `x` for group would be. On a *directory*, files created inside inherit the directory's group. Useful for shared team folders.
- **sticky bit** — `t` where `x` for other would be. In a shared writable dir like `/tmp`, only the *file's owner* can delete a file, even if others have write on the directory. This is why `/tmp` is safe for everyone to share.

### Hands-on 2: prove the model in a container

Spin up a fresh Ubuntu container and run through the sequence:

```bash
docker run --rm -it ubuntu:24.04 bash
```

Inside the container, run these in order:

```bash
# --- 1. Who am I? What's my umask? ---
whoami
id
umask

# --- 2. Create a file, inspect its default perms ---
touch /tmp/hello.txt
ls -l /tmp/hello.txt

# --- 3. Change perms with both forms of chmod ---
chmod 600 /tmp/hello.txt
ls -l /tmp/hello.txt        # expect: -rw-------
chmod g+r,o+r /tmp/hello.txt
ls -l /tmp/hello.txt        # expect: -rw-r--r--

# --- 4. Create a non-root user, then become her ---
useradd -m alice
id alice
ls -ld /home/alice
su - alice

# --- 5. As alice, see the world with narrower privilege ---
whoami
id
ls -la /root 2>&1 | head -5     # root's home is 700 — should be denied
touch /etc/alice-was-here 2>&1  # /etc is root-owned — should fail
touch ~/notes.txt && ls -l ~/notes.txt   # her own home is fine
exit                              # back to root

# --- 6. Give a file to alice with chown ---
touch /tmp/gift.txt
ls -l /tmp/gift.txt
chown alice:alice /tmp/gift.txt
ls -l /tmp/gift.txt

exit   # exit container
```

**Paste back these five lines/blocks:**

1. Output of `id` (as root inside the container).
2. `ls -l /tmp/hello.txt` **after the first `chmod 600`** — confirms `-rw-------`.
3. The error(s) alice got trying to read `/root` and write to `/etc`.
4. The `ls -l ~/notes.txt` line as alice — confirms she can write in her own home.
5. The `ls -l /tmp/gift.txt` line **after `chown alice:alice`** — confirms ownership transferred.

Once we have your outputs, the hands-on log gets updated and we move to **Concept 3 — processes & services** (`ps`, `top`, `systemctl`, `journalctl` — the toolkit you use to answer *"what's actually running on this box?"*).

### Concept 2 — takeaway

- 🔑 Every "Permission denied" is answered by the same three-question sequence: *who am I → who owns the file → what do the bits say?*
- 🔢 The 9 permission bits are the whole game: 3 sets of `rwx` for owner, group, other. Octal is just those bits written compactly.
- 📂 On a **directory**, `x` means "the key to the door" — without it, files inside are unreachable even if their own bits say yes.
- 🧨 **Root ignores the 9-bit check.** That's why running containers or K8s pods as root is a security anti-pattern.
- 🎭 `umask` decides the *default* perms new files get. Security-sensitive systems set it to `077` (private-by-default).
