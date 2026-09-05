# Module 00 — Hands-on log

> *This log captures the actual commands run during the setup session, the outputs they produced, and — most importantly — the lessons learned from surprises. If a command misbehaved, it stays in the log with the fix, because that's more educational than a clean-room replay.*

## Session 1 — day one of the journey

### 1. Inventory the workstation

**Ran:**

```bash
which az terraform docker kubectl helm ansible packer gh git
az --version
```

**Result:** `az`, `terraform`, `docker`, `kubectl`, `helm`, `gh`, `git` were present. `ansible` and `packer` were missing. Azure CLI reported version 2.87.0.

**Lesson:** Take five minutes at the start of any cloud/DevOps engagement to inventory your workstation. Every missing tool discovered *now* is a known unknown instead of a mid-lab surprise.

### 2. Install the missing tools (macOS)

**Ran:**

```bash
brew install ansible
brew trust hashicorp/tap        # one-time consent for HashiCorp's third-party tap
brew install packer
packer version
```

**Result:** Ansible installed cleanly. Packer installed at version `1.16.0` after the trust prompt.

**Lesson:** Homebrew introduced a "tap trust" gate in recent versions — third-party taps must be explicitly trusted the first time. HashiCorp's tap (`hashicorp/tap`) is the official one that ships both `terraform` and `packer`; trusting it once is safe and permanent.

### 3. Discover your Azure access (the real state)

Started with what seemed like a simple check:

```bash
az account show
```

That returned a subscription in an unexpected tenant. The CLI's cache was showing stale entries. The `--refresh --all` combo reveals the truth:

```bash
az account list --refresh --all --output table
```

This surfaced two important facts: one previously-cached subscription no longer had an accessible role assigned; and another tenant required MFA to reach. Cached lists lie — always refresh at the start of a new session.

**Lesson:** `az account list` shows only cached sessions. `--refresh` round-trips to Azure; `--all` includes cross-tenant subs. Always run the fully-decorated version first on a new machine or after any auth change.

### 4. Log into the tenant that owns the sub you want to use

**Ran:**

```bash
az login --tenant <tenant-id>
```

The `--tenant` flag pins the login to a specific directory. Without it, `az login` uses your home tenant — which is often *not* the tenant that owns the subscription you actually want to work in.

**Lesson:** In a multi-tenant world (very common in enterprises and even for individuals with a personal + work Microsoft account), the `--tenant` flag is essential every time you `az login`.

### 5. Discover your actual RBAC

The obvious command failed:

```bash
az role assignment list --assignee <you@example.com>
```

...with `Insufficient privileges to complete the operation` — because the tenant blocks Graph directory reads for regular users. The workaround skips Graph by querying with the object id directly. Any failed write command in Azure conveniently prints your object id in the error message; grab it from there:

```bash
az role assignment list \
  --assignee-object-id <your-object-id> \
  --all --output table
```

One row came back for this account:

```
Principal                                     Role   Scope
────────────────────────────────────────────  ─────  ────────────────────────────────────────
chishty_niftyitsolution.com#EXT#@example.com  Owner  /subscriptions/…/resourceGroups/niftyexp
```

**Lesson:** RBAC errors don't always mean "no role." Sometimes they mean *"the tenant is blocking your directory read."* Use `--assignee-object-id` to sidestep Graph. And read the `Scope` column carefully: Owner scoped to a specific resource group is *not* Owner scoped to a subscription.

### 6. Confirm the boundary by attempting a write

Two safe writes attempted (both expected to fail, given the RBAC finding above):

```bash
az provider register --namespace Microsoft.ContainerService
az group create --name learn-permcheck --location eastus2
```

Both returned `AuthorizationFailed` with the exact action strings:

```
Microsoft.ContainerService/register/action
Microsoft.Resources/subscriptions/resourcegroups/write
```

Those action strings are gold — copy them verbatim into any admin unlock request (see [`asking-for-cloud-access.md`](./asking-for-cloud-access.md)).

**Lesson:** When you're not sure what a permission level covers, attempt the write on a throwaway resource and read the error. The error message names the exact action string, which is the currency of admin-permission conversations.

### 7. Choose a safe region + a naming discipline

Since Owner on this account is scoped to one RG (`niftyexp` in `eastus2`), the default lab region for every subsequent module is **`eastus2`** — the same region that RG lives in. Every lab resource going forward is:

- Named with a `learn-` prefix
- Tagged `owner=<you>` and `purpose=learning`
- Torn down at the end of the session

This discipline keeps a shared cloud account safe from accidental collisions.

### 8. Commit the scaffolding

Repo initialised, README + Module 00 docs + `.gitignore` created, first commit made, published to GitHub. The scaffold *is* durable state — any future session (yours or a collaborator's) can resume from the repo alone.

---

## What you take away from Module 00

- ✅ You know the story every Cloud/DevOps tool fits into.
- ✅ Your workstation has every CLI you'll need through Module 10.
- ✅ You know exactly what your cloud account can and cannot do.
- ✅ You have a naming and tagging discipline for a shared account.
- ✅ You have a template for asking an admin to unblock what you need.

Next: [Module 01 — Linux & Shell for DevOps](../01-linux-shell/README.md).
