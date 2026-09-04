# Module 00 — Setup & The Big Picture

> *You already ship software. This weekend, you learn to build the road the software runs on.*

## The story

Every job posting for "Cloud / DevOps Engineer" reads like a menu of unrelated tools — Terraform, Kubernetes, Helm, Ansible, Packer, GitHub Actions, Jenkins, AWS, GCP. It isn't. There's a single story running through all of them, and once you see the story, each tool falls into a labelled slot in your head instead of floating around as jargon.

Here's the story in one paragraph.

A team has an application. Someone has to give that application **a place to run** (a server, a container, a cluster of containers). Somebody has to **describe that place in code** so anyone can rebuild it, so it can be version-controlled, so it can be code-reviewed. Somebody has to **install the software onto that place** — repeatably, without hand-editing config files at 2am. Somebody has to build a **conveyor belt** that takes a git push and delivers a running new version to real users, on its own. And somebody has to **watch** the whole system, because in production, what you cannot see will hurt you.

Every tool on the job posting maps to one part of that story:

| Part of the story | What the tool is | Examples on the posting |
|---|---|---|
| The place where things run | **The cloud** | AWS, Azure, GCP |
| Describing that place in code | **Infrastructure as Code (IaC)** | Terraform |
| Packaging the app so it runs identically everywhere | **Containers** | Docker |
| Orchestrating hundreds of containers so they self-heal and scale | **Container orchestration** | Kubernetes, EKS (AWS), AKS (Azure), GKE (GCP) |
| Packaging a Kubernetes app so it's installable in one command | **Package manager for K8s** | Helm |
| Configuring servers (installing packages, editing files, starting services) — repeatably | **Configuration management** | Ansible |
| Baking pre-configured machine images so servers boot ready-to-run | **Image build** | Packer |
| The conveyor belt from `git push` to production | **CI/CD** | GitHub Actions, Jenkins |
| Watching what production is doing | **Observability** | Prometheus, Grafana, CloudWatch, Azure Monitor |

The underlying skills — Linux, shell scripting, networking, security/IAM, troubleshooting — are the substrate everything else stands on.

Your job as a Cloud/DevOps engineer is to own all of that, not necessarily as an expert in each tool, but as somebody who can navigate the whole story and reach for the right tool at the right time.

---

## Where you already are

You are not starting from zero. Today you:

- **Write code and ship it to Azure** with a proper CI/CD pipeline. That means you already understand *what* CI/CD does — you've seen `git push` become a running deployment. What's new for you is the *infra side* of that same pipeline: how the cloud got provisioned, how the runners got authenticated, how the cluster or VM was configured, how monitoring was wired up.
- **Have followed release procedures** in a team. You know that "just push it" is what a jr does; "push it after checking the changelog, the migration, the rollout plan, the observability dashboard" is what senior does. That instinct carries over.
- **Have Azure portal fluency**. You've clicked around, seen resource groups, App Services, key vaults. You'll now do the same things from the CLI and from Terraform, and the mapping will feel natural.

So the pitch of every future module is: *"You already understand X for apps. Here's X from the infra side."*

---

## The three clouds at a glance

You'll be interviewed on AWS + GCP, but you'll be practicing on Azure. That's fine — the concepts are 90% identical, only the names change. Here's the cheat-sheet you memorize once:

| Concept | AWS | Azure | GCP |
|---|---|---|---|
| Compute (VMs) | EC2 | Virtual Machine | Compute Engine |
| Managed Kubernetes | EKS | AKS | GKE |
| Container registry | ECR | ACR | Artifact Registry |
| Object storage | S3 | Blob Storage | Cloud Storage |
| Virtual network | VPC | VNet | VPC |
| Subnets | Subnets (per AZ) | Subnets | Subnets (regional) |
| Load balancer (L4) | NLB | LB (Standard) | Network LB |
| Load balancer (L7) | ALB | Application Gateway | HTTPS LB |
| DNS | Route 53 | Azure DNS | Cloud DNS |
| Managed database | RDS | Azure SQL / Cosmos | Cloud SQL / Spanner |
| Secrets | Secrets Manager | Key Vault | Secret Manager |
| Identity + roles | IAM | Entra ID + RBAC | Cloud IAM |
| CI-friendly identity | IAM Role (with OIDC) | Managed Identity / Workload Identity | Service Account (with Workload Identity Federation) |
| Serverless functions | Lambda | Functions | Cloud Functions |
| Monitoring | CloudWatch | Azure Monitor | Cloud Monitoring |
| Logs | CloudWatch Logs | Log Analytics | Cloud Logging |
| Infra-as-code (native) | CloudFormation / CDK | ARM / Bicep | Deployment Manager |
| Infra-as-code (cross-cloud) | Terraform | Terraform | Terraform |

Cross-cloud IaC via Terraform is why Terraform is *the* skill on this posting — it's the one lever that works everywhere.

---

## Your workstation, verified

Today's session verified the following:

| Tool | Status | Version / note |
|---|---|---|
| `az` (Azure CLI) | ✅ | 2.87.0 |
| `terraform` | ✅ | via HashiCorp tap |
| `docker` | ✅ | Docker Desktop |
| `kubectl` | ✅ | ready |
| `helm` | ✅ | ready |
| `git` | ✅ | system git |
| `gh` (GitHub CLI) | ✅ | logged in as `chishty313` |
| `ansible` | ✅ | installed today |
| `packer` | ⏳ | needs `brew trust hashicorp/tap && brew install packer` |
| `kind` | ⏳ | will install in Module 06 |

`az login --tenant 69af0553-f650-480d-967d-1ed003fdef20` is the command that gets you into the right Azure tenant; MFA is required.

---

## Your cloud access, honestly

This is the section I wrote first, because you have to know what ground you're standing on before you take a step. Details are in [`cloud-account-status.md`](./cloud-account-status.md). The one-line version:

> **Owner on the `niftyexp` resource group. Reader on the whole subscription. Nothing else.**

That means every lab resource we create lives inside `niftyexp`, named `learn-*`, tagged `owner=chishty purpose=devops-prep`. Nothing outside that RG. Nothing untagged. And after each session we tear down what we created (see the ground rules in the [repo README](../../README.md)).

Three Azure services on the syllabus (**AKS, ACR, Key Vault**) need a sub-level provider register before we can use them — which only sub-level Contributor can do. If the admin unlocks that this weekend, great. If not, we substitute cleanly: **kind** (Kubernetes-in-Docker) replaces AKS, **GHCR** (GitHub Container Registry) replaces ACR, and encrypted `.env.local` + GitHub Actions secrets replace Key Vault for lab exercises. The concepts and the code we write are identical either way.

The email you send to the admin to try to unlock it is drafted in [`admin-request-template.md`](./admin-request-template.md).

---

## What Module 01 will do

Module 01 is Linux + Shell for DevOps. You already know the basics; what we cover is the *DevOps flavour* — the specific 20% of Linux that shows up in 80% of on-call incidents and interview whiteboards:

- The filesystem hierarchy that actually matters (`/etc`, `/var/log`, `/proc`, `/sys`)
- Permissions and ownership (why your container runs as root and why that's bad)
- Processes and services (`systemd` because every modern Linux server uses it)
- Networking from the command line (`ss`, `curl`, `dig`, `nc`, `tcpdump`)
- The pipeline (`grep`, `sed`, `awk`, `find`, `xargs`, `cut`, `sort`, `uniq`) — the DevOps interview loves these
- Bash scripting patterns that don't fall over the first time you use them in production (`set -euo pipefail`, trap, exit codes, function returns)
- `ssh` and friends (`scp`, `rsync`, ssh config, jump hosts)

Hands-on for Module 01 is done on your Mac terminal — no cloud needed. That means Module 01 is unblocked by any access issue and we can start immediately after this setup is committed.

---

## Cadence for the weekend

- **Friday (today, ~8h available):** Modules 00, 01, 02, 03 — foundations. Ends with you having a small VNet+VM built and understood.
- **Saturday (~8h):** Modules 04, 05, 06 — Terraform depth, Docker deep-dive, Kubernetes fundamentals on `kind`.
- **Sunday (~8h):** Modules 07–12 — AKS-mental-model, Helm, CI/CD end-to-end, Ansible + Packer, monitoring, cross-cloud translation.
- **Monday onward (weekday evenings):** Module 13 — interview prep, more depth on any module you feel wobbly on.

Ready. Once this module is committed, we start Module 01.
