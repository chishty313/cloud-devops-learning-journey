# Cloud account status — as of 2026-09-04

I'm writing this section down because "what can I actually do on my cloud account" is a decision-shaping fact for every subsequent module. Six months from now, the details will drift; this document is a snapshot of the moment I started.

## Azure — the accounts I have access to

I signed in with `chishty@niftyitsolution.com`. The Azure CLI shows me three subscriptions across three tenants:

| Subscription | Sub id | Tenant | State | My access |
|---|---|---|---|---|
| Azure subscription 1 (working one) | `56397b98-2390-4ae1-85d3-463f7c64f60b` | Default Directory (`Jumadancun94gmail.onmicrosoft.com`, tenant id `69af0553-…-fdef20`) | Enabled | **Owner on RG `niftyexp` only. Reader on the sub.** |
| Azure subscription 1 (old personal) | `1e87da18-…-31f6ad` | `RiyaGupta89outlook.onmicrosoft.com` | Disabled | Was Owner, but sub is currently disabled. |
| Microsoft Azure Sponsorship lvl 2 | `b06c8871-…-788dbf` | Nifty IT Solution LTD (`niftyitsolution.com`) | Phantom (in old cache; not actually accessible) | None. Being *invited to a tenant* is not the same as *being granted a role on a subscription in that tenant.* |

The tenant name `Jumadancun94gmail.onmicrosoft.com` is where the "jumadan…" I half-remembered was coming from — it's the default domain of a directory created by someone whose Google email was `Jumadancun94@gmail.com`. It's the "personal Azure sub" my colleague set up.

MFA is required for that tenant. `az login --tenant 69af0553-f650-480d-967d-1ed003fdef20` triggers the browser flow.

## The exact RBAC I have

I asked Azure what roles are assigned to my object id (`ccbd63d8-88c0-4bba-b749-5b9393052ede`) across every scope in the working sub:

```
az role assignment list \
  --assignee-object-id ccbd63d8-88c0-4bba-b749-5b9393052ede \
  --all --output table
```

One row came back:

```
Principal                                                           Role   Scope
------------------------------------------------------------------  -----  ---------------------------------------------------------------------------
chishty_niftyitsolution.com#EXT#@Jumadancun94gmail.onmicrosoft.com  Owner  /subscriptions/56397b98-…/resourceGroups/niftyexp
```

That "Owner on `/subscriptions/…/resourceGroups/niftyexp`" line is my entire universe on this sub. I can:

- Do anything **inside** `niftyexp` — as long as the resource type's provider is registered at sub level.
- Read (list) other RGs and sub-level metadata.

I cannot:

- Create a new RG.
- Register a subscription-level resource provider.
- Grant roles.
- Touch anything in the other RGs (`niftyai`, `jenkins-rg`, `Toastmaster`, `AI-SPEECH-RG`, `rg-Jumadancun94-7881`, and the rest).

## Resource providers registered on this sub

Registered (so I can create these inside `niftyexp`):

- `Microsoft.Compute` — VMs, VMSS, disks
- `Microsoft.Network` — VNet, subnets, NSG, LB, public IPs
- `Microsoft.Storage` — storage accounts (blob/file/queue/table)
- `Microsoft.OperationalInsights` — Log Analytics workspaces

Not registered (so I cannot create these anywhere on this sub, even inside `niftyexp`, until an admin registers them):

- `Microsoft.ContainerService` — **AKS**
- `Microsoft.ContainerRegistry` — **ACR**
- `Microsoft.KeyVault` — **Key Vault**

## Substitutes for the blocked services

The concepts and the code you write are the same; only the underlying platform changes.

| Blocked Azure service | Substitute I use for labs | Why it's fine |
|---|---|---|
| AKS | `kind` (Kubernetes-in-Docker) locally, and the Terraform module you write is `azurerm_kubernetes_cluster`-shaped so it drops onto Azure the moment the provider is registered. | Same Kubernetes API. Same manifests. Same `kubectl` and `helm` workflows. |
| ACR | GitHub Container Registry (GHCR, `ghcr.io/chishty313/…`) | Standard modern CI/CD registry. What the industry moved to anyway. |
| Key Vault | `.env.local` files (gitignored) for local runs, GitHub Actions encrypted secrets for pipelines | Enough for learning; production-shaped secret retrieval is a 20-line module-writing exercise once KV is unblocked. |

## Region choice: East US 2

`niftyexp` lives in `eastus2` (I checked with `az group show --name niftyexp`). Since I'm Owner *only* on this RG, all lab resources have to live in the same RG, which means the same region. So the default region for every lab in this repo is **`eastus2`**. When you see `--location eastus2` throughout the repo, that's why.

The RG already contains **Cognitive Services** and an **AI Foundry project** — those are the previous work in `niftyexp`. Do not touch or delete them. Every resource we create carries `owner=chishty purpose=devops-prep` so `az resource list -g niftyexp --tag purpose=devops-prep` cleanly separates ours from theirs at cleanup time.

## VM quota

- East US 2, BS family: check with `az vm list-usage --location eastus2 -o table`.
- Lab VMs use `Standard_B2s` (2 vCPUs) so a 10-vCPU quota holds 5 concurrent VMs — plenty.

## What I'm asking the admin to do (draft)

See [`admin-request-template.md`](./admin-request-template.md).
