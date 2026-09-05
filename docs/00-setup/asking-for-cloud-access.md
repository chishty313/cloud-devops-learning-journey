# Asking for cloud access — a template that respects everyone's time

> *When you need a cloud admin to unblock you, the difference between a five-minute unblock and a five-day back-and-forth is how clearly you asked. Here is a template that works.*

## The three principles behind a good ask

1. **Say exactly which subscription/project and what action.** Not "I need Azure access" — give the sub id and the action string.
2. **Offer the smallest permission that unblocks you.** Admins are more willing to grant one thing than five, and more willing to grant scoped than sub-wide.
3. **Include a graceful fallback.** "If you can't do X, I'll do Y locally" makes it clear you're not stuck waiting on them.

## Template (Azure flavour)

Copy, adapt, send.

---

**Subject:** Quick Azure unblock — provider register (30-second CLI)

Hi,

I'm working on Cloud/DevOps learning on the shared subscription `<subscription-name>` (`<subscription-id>`). I'm currently Owner on the `<your-rg>` resource group and I'll keep everything inside it (prefixed `learn-*`, tagged `owner=<you> purpose=learning`, torn down at end of each session).

Three things would unblock the harder labs (AKS, ACR, Key Vault). In order of easiest to most useful:

1. **Register three resource providers** on the sub (30 seconds, no cost, no impact on existing resources):

   ```bash
   az provider register --namespace Microsoft.ContainerService --subscription <sub-id>
   az provider register --namespace Microsoft.ContainerRegistry --subscription <sub-id>
   az provider register --namespace Microsoft.KeyVault --subscription <sub-id>
   ```

2. *(Nice to have)* **Grant me `Contributor`** on the subscription. Blast radius is limited by our tagging + prefix discipline. Portal path: `Subscriptions → <sub-name> → Access control (IAM) → Add role assignment → Contributor → <me@example.com>`.

3. *(Alternative to #2 if you'd rather keep sub-level tight)* just register the providers in step 1, and I'll continue working inside my existing RG.

If neither is possible this week, no worries — I'll do the Kubernetes labs locally on `kind` (Kubernetes-in-Docker) and use GitHub Container Registry instead of ACR. Loop back when you have a minute.

Thanks!

---

## AWS variant

The same three principles apply. The specific asks change:

- **Instead of "register a provider"**: ask for `iam:PassRole` on a specific role, or for a service-linked role to be created.
- **Instead of "Contributor on the sub"**: ask for an IAM group membership, or for a specific policy attached to your user.
- **Instead of "Owner on an RG"**: ask for a specific IAM boundary policy that scopes what your role can touch.

Rough template:

> *I need to run `<service>` labs (e.g., EKS). Could you attach `AmazonEKSClusterPolicy` to my IAM user `<me>`, and add me to the `eks-learners` IAM group if one exists? I'll only create resources tagged `owner=<me> purpose=learning`, and I'll tear them down after each session. If you'd rather scope tighter, an IAM boundary that limits me to a single VPC or tag namespace would be fine.*

## GCP variant

- **Instead of "register a provider"**: ask for services to be enabled on the project: `gcloud services enable container.googleapis.com`.
- **Instead of "Contributor"**: ask for a scoped role at the project level, e.g. `roles/container.admin` for GKE, `roles/artifactregistry.admin` for Artifact Registry.

Rough template:

> *I need to run GKE labs on project `<project-id>`. Could you enable `container.googleapis.com` and grant me `roles/container.admin` scoped to the project? I'll tag everything `owner=<me> purpose=learning` and delete resources after each session.*

## What NOT to do

- ❌ "Please give me full admin, I'll be careful." Admins reflexively refuse; you'll wait weeks.
- ❌ "I got an AuthorizationFailed error." Paste the *entire* error including the action string; without it the admin has to guess.
- ❌ Ask via a channel the admin ignores (broadcast Slack, comments on a wiki page). Use whatever's fastest: DM, ticket, or in person.

## After they grant it

Verify from your side within a minute:

```bash
# Azure
az account clear && az login --tenant <tenant-id>
az provider list --query "[?namespace=='Microsoft.ContainerService'].registrationState"

# AWS
aws sts get-caller-identity
aws iam list-attached-user-policies --user-name <me>

# GCP
gcloud services list --enabled --project <project-id> | grep container
```

Reply to confirm, so the admin knows they can close the loop. Small courtesies compound.
