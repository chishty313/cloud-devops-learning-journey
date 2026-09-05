# Module 01 — Hands-on log

> *A running log of the actual commands run, the outputs they produced, and the lessons drawn from surprises.*

## Session 1 · Concept 1 — The filesystem hierarchy

### Setup — spin up a real Linux container

```bash
docker run --rm -it ubuntu:24.04 bash
```

The first time this runs, Docker pulls the image:

```
Unable to find image 'ubuntu:24.04' locally
24.04: Pulling from library/ubuntu
Digest: sha256:33ceb71981b602c1a7443a53469e4dba065f7503eab3078a2d7a57a2ab987517
Status: Downloaded newer image for ubuntu:24.04
```

Landed in `root@<container-id>:/#`. Real Ubuntu, on the same machine, in seconds. This is why containers changed the world.

### Exploration commands

**`ls /`** — the root of the tree.

```
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

Every directory from the module doc's tree is there. Notice the container has the full Linux directory convention even though it runs almost no services — the convention is *"you always ship this shape, even empty."*

**`ls /etc | head -20`** — a peek at system config.

```
alternatives    apt            bash.bashrc    bindresvport.blacklist    cloud
cron.d          cron.daily     debconf.conf   debian_version           default
dpkg            e2scrub.conf   environment    fstab                    gai.conf
gnutls          group          group-         gshadow                  gshadow-
```

Even this bare image has meaningful config: `apt/` for package manager sources, `group`/`gshadow` for group membership, `fstab` for mounts. On a production server, `/etc` has *hundreds* of entries.

**`cat /etc/os-release`** — the machine-readable OS identity.

```
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```

Every distro ships this file; scripts should read `ID` (short form) and `VERSION_ID` (numeric) when they need to branch by distro.

**`cat /proc/meminfo | head -5`** — live kernel memory state.

```
MemTotal:        4010356 kB
MemFree:         1006524 kB
MemAvailable:    2770236 kB
Buffers:          972468 kB
Cached:           895348 kB
```

`MemTotal` here is ~4 GB — that's what the container's cgroup exposes. `MemAvailable` (the field that matters for "can I allocate more?") is roughly `Free + Cached + Buffers`, which is why apps that go by `Free` alone tend to over-report a problem.

**`cat /proc/cpuinfo | grep '^model name' | head -1`** — CPU model.

The `grep` returned **empty**.

> [!IMPORTANT]
> **This is the ARM gotcha promised in the module doc.**
> On an Apple Silicon Mac, Docker Desktop runs an ARM64 Linux VM. ARM `/proc/cpuinfo` doesn't have a `model name:` line — it has fields named `CPU implementer`, `CPU architecture`, `CPU variant`, `CPU part`. So an x86-shaped grep pattern silently returns nothing.
>
> **The portable version:** `cat /proc/cpuinfo | head -20` (just look at the raw output) or `grep -iE 'model|cpu|processor' /proc/cpuinfo | head`. When writing cross-platform scripts, never assume the format of `/proc/cpuinfo` — always look at it first on the arch you're targeting.

**`ls /var/log`** — bare-image log dir.

```
alternatives.log  apt  bootstrap.log  btmp  dpkg.log  faillog  lastlog  wtmp
```

Sparse — no services running yet. Three of these are binary and interesting:

| File          | What it records                | Read with          |
| ------------- | ------------------------------ | ------------------ |
| `btmp`        | Failed login attempts          | `lastb`            |
| `wtmp`        | Successful logins/logouts      | `last`             |
| `lastlog`     | Last login per user            | `lastlog`          |

These are what security tools like `fail2ban` watch to detect brute-force SSH.

### Cleanup

```bash
exit
```

Container exits, `--rm` removes it. `docker ps -a` confirms no residue.

### Lessons banked

1. `docker run --rm -it <image> bash` is the fastest way to get a scratch Linux to poke at.
2. Every Linux distro ships the same directory conventions — `/etc`, `/var/log`, `/proc` etc. — even if they're empty.
3. `/proc` is a *live window into the kernel*, not a set of static files. Most system-inspection tools are pretty-printers over `/proc`.
4. **Tool output formats are architecture-specific.** An x86-shaped `grep` pattern will silently miss on ARM64. Always look at raw output before wrapping it in a script.
5. `MemAvailable` (not `MemFree`) is the correct "how much can I allocate?" metric.
6. `/var/log/btmp` and `/var/log/wtmp` are the audit trail for logins; know how to read them (`lastb`, `last`).

Concept 2 (permissions & ownership) begins next.
