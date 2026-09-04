# Module 00 — Hands-on log

*What I actually ran, what came back, and what I learned from each command. This file is written up during the session, not after — it's the running lab notebook.*

## Session 1 — 2026-09-04

### 1. Toolbox check

**What I ran:**

```bash
which az terraform docker kubectl helm ansible packer gh git
az --version
```

**Result:** `az`, `terraform`, `docker`, `kubectl`, `helm`, `gh`, `git` all present. `ansible` and `packer` missing. Azure CLI is 2.87.0.

**Lesson:** Before you begin any cloud work, one command to inventory the workstation saves half an hour of confusion later. Any missing tool now is a known unknown, not a surprise mid-lab.

### 2. Fixing the missing tools

**What I ran:**

```bash
brew install ansible
brew trust hashicorp/tap
brew install packer
```

**Why `brew trust` for packer:** Homebrew recently added a trust prompt for third-party taps. HashiCorp's tap is the *official* one that publishes both `terraform` and `packer`, so trusting it is safe and permanent — one-time step per tap.

### 3. Discovering my Azure access, the hard way

I ran what I *thought* would be a simple sanity check:

```bash
az account show
```

It came back with a subscription in a tenant I didn't recognise (`RiyaGupta89outlook.onmicrosoft.com` — my old personal). So I ran:

```bash
az account list --output table
az account list --refresh --all --output table
```

This surfaced two more things: a **cached ghost** entry ("Microsoft Azure Sponsorship lvl 2") in the Nifty tenant that turned out to have no accessible subscription attached to me, and a warning that another tenant (`69af0553-…-fdef20`) needed MFA. That last tenant is where the sub I could actually use lives.

**Lesson:** `az account list` shows only *cached* sessions. `--refresh --all` is the truth-teller. Always run it first on a new machine or after any auth change.

### 4. Logging into the right tenant

**What I ran:**

```bash
az login --tenant 69af0553-f650-480d-967d-1ed003fdef20
```

Browser opened, MFA challenge, back in. Now `az account list --output table` shows the working sub: `Azure subscription 1` (id `56397b98-…-c64f60b`), tenant `Jumadancun94gmail.onmicrosoft.com`, enabled.

**Lesson:** `az login --tenant <tenant-id>` is how you jump between directories. Without `--tenant` you land in whatever the CLI decided is your home tenant, which may not be the one that owns the sub you want.

### 5. Finding out what I actually have permission to do

**What I ran (and learned to run):**

Bare `az role assignment list --assignee <email>` failed with a Graph "insufficient privileges" error, because the tenant blocks directory reads for regular users. Fine — Graph isn't the only way. The by-object-id path skips Graph:

```bash
az role assignment list --assignee-object-id ccbd63d8-88c0-4bba-b749-5b9393052ede --all --output table
```

That returned exactly one row: **Owner on `/subscriptions/56397b98-…/resourceGroups/niftyexp`**. Sub-wide I'm only Reader.

**Lesson:** RBAC failures don't always mean you have no role — sometimes they mean *the tenant is blocking your directory read.* Use `--assignee-object-id` to sidestep Graph and query RBAC directly. Also: Owner on an RG is not Owner on a sub. Never assume portal breadcrumbs mean sub-level.

### 6. Confirming what I couldn't do

I tried to register the three missing providers and to create a scratch RG. All failed with `AuthorizationFailed` — write on sub scope is not granted. That's the boundary of what Reader can do.

**Lesson:** The fastest way to learn what a permission level *actually* covers is to attempt the write on a throwaway resource and read the error. The error message names the exact action string (`Microsoft.Resources/subscriptions/resourcegroups/write`) — which is what you paste into an admin request.

### 7. Locking in the working state

Wrote this doc, [`../00-setup/README.md`](./README.md), [`cloud-account-status.md`](./cloud-account-status.md), the [`admin-request-template.md`](./admin-request-template.md), the root [`README.md`](../../README.md), and `.gitignore`. Committed on `main`. Pushed to GitHub.

---

## Session N — <date> <heading>

*(Future sessions append here, same shape: what I ran, what it returned, what I learned. Each becomes its own dated section.)*
