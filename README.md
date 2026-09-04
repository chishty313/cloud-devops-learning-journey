# Cloud / DevOps Engineer — Interview Prep

**Target role:** Cloud / DevOps Engineer at Together (i2gether.com), Dhaka.
**Author:** Chishty (jr software engineer today; shipping to Azure with CI/CD; growing into infra).
**Method:** Storytelling documentation, hands-on execution, everything committed.

---

## Why this repo exists

The job posting asks for AWS + GCP, Terraform, Docker, Kubernetes/EKS, Helm, Ansible, Packer, CI/CD (GitHub Actions / Jenkins), Linux, Shell, networking, security, IAM, monitoring — 2–4 years of it. I have ~1 year of application-side experience shipping to Azure with proper CI/CD, and I know pieces of everything on that list. This repo is me going from *"I know pieces"* to *"I can walk any interviewer through the full picture with a real project to point at."*

Every module in this repo:
1. **Tells a story** — why the tool exists, what problem it solves, how it fits the bigger picture.
2. **Is hands-on** — I actually run every command on my machine; the exact steps + outputs live in `docs/NN-*/hands-on.md`.
3. **Is a permanent reference** — a future me should be able to reopen this repo in a year and not have to Google anything to remember what I did or why.

The hands-on runs on **Azure** (because that's the cloud access I have). The concepts port to AWS and GCP — a cross-cloud translation sheet is Module 12.

---

## Current status

| # | Module | Status | When |
|---|---|---|---|
| 00 | Setup & The Big Picture | 🟢 Started | Fri 2026-09-04 AM |
| 01 | Linux & Shell for DevOps | ⚪ Planned | Fri 2026-09-04 |
| 02 | Networking Foundations | ⚪ Planned | Fri 2026-09-04 |
| 03 | IAM & Security | ⚪ Planned | Fri 2026-09-04 |
| 04 | Terraform Deep Dive | ⚪ Planned | Sat 2026-09-05 |
| 05 | Docker Fundamentals + Advanced | ⚪ Planned | Sat 2026-09-05 |
| 06 | Kubernetes Core | ⚪ Planned | Sat 2026-09-05 |
| 07 | AKS / EKS mental model | ⚪ Planned* | Sun 2026-09-06 |
| 08 | Helm | ⚪ Planned | Sun 2026-09-06 |
| 09 | CI/CD: GitHub Actions + Jenkins | ⚪ Planned | Sun 2026-09-06 |
| 10 | Ansible + Packer | ⚪ Planned | Sun 2026-09-06 |
| 11 | Monitoring, Logging, Troubleshooting | ⚪ Planned | Sun 2026-09-06 |
| 12 | Cross-Cloud Translation (Azure ↔ AWS ↔ GCP) | ⚪ Planned | Sun 2026-09-06 |
| 13 | Interview Prep (behavioral + technical + comp-programming angle) | ⚪ Planned | Mon 2026-09-07+ |

`*` Module 07 runs on local `kind` (Kubernetes-in-Docker) unless the Azure sub admin registers `Microsoft.ContainerService` in time; the concept and the code are the same.

Legend: 🟢 in progress · ✅ done · ⚪ planned · 🟡 blocked

---

## Repo layout

```
README.md                    ← this file: index, syllabus, progress
docs/
  00-setup/
    README.md                ← the story of Module 00
    cloud-account-status.md  ← what my Azure access actually is (2026-09-04)
    admin-request-template.md← what to send to sub admin to unlock AKS/ACR/KV
    hands-on.md              ← every command I ran and what it returned
  01-linux-shell/…
  …
labs/
  NN-<topic>/                ← real code (Terraform, Helm, Ansible, Packer, workflows)
cheatsheets/
  <tool>.md                  ← one-page cram sheets per tool
```

---

## Ground rules I set myself

1. **No lab resource lives outside the `niftyexp` RG** on Azure. That RG is my sandbox because it's the only scope I'm Owner on. Every resource name starts with `learn-`. Every resource carries tags `owner=chishty purpose=devops-prep`.
2. **After every hands-on session, tear it down.** `az resource list -g niftyexp --tag purpose=devops-prep -o table` shows what I created; delete each one before closing the laptop.
3. **Commit after every module.** Commit message describes what I *learned or built*, not "update".
4. **If it's not in this repo, it didn't happen.** No side notes, no separate scratch files that vanish.

---

## Job posting (verbatim, saved for reference)

> Job title: **Cloud / DevOps Engineer**
> Requirements:
> - 2–4 years' experience in Cloud/DevOps with AWS & GCP knowledge
> - Hands-on with Terraform, Docker, Kubernetes/EKS, Helm, Ansible & Packer
> - Experience with Git, Linux, Shell scripting & CI/CD (GitHub Actions/Jenkins)
> - Strong knowledge of networking, security, IAM, monitoring & troubleshooting
> - Experience in competitive programming and coding contests
>
> Open positions: 04 · Location: Dhaka, Bangladesh · CV to: job@i2gether.com · Deadline: 06 Sep 2026.
