# Admin unlock request — draft

Send this to whoever admins subscription `56397b98-2390-4ae1-85d3-463f7c64f60b` on the `Jumadancun94gmail.onmicrosoft.com` tenant (likely `Jumadancun94@gmail.com` or whoever set that Azure account up). Slack/WhatsApp/email — whichever channel is fastest.

The minimum ask is the two provider registers; the ideal ask is Contributor on the sub. Both take under a minute.

---

**Subject:** Quick Azure ask — unblock DevOps learning labs

Hi,

I'm using the shared Azure sub `Azure subscription 1` (`56397b98-2390-4ae1-85d3-463f7c64f60b`) for a weekend of self-study for a Cloud/DevOps role I'm interviewing for. I'm Owner on the `niftyexp` RG (thanks!) which is where all my learning resources will live — I'll prefix everything `learn-*`, tag everything `owner=chishty purpose=devops-prep`, and tear it all down at the end of each session so nothing touches your existing workloads.

Three things would unblock the harder labs (AKS, ACR, Key Vault). In order of easiest to most useful:

1. **Register three resource providers** on the sub (30-second CLI, no cost, no impact on existing resources):
   ```bash
   az provider register --namespace Microsoft.ContainerService --subscription 56397b98-2390-4ae1-85d3-463f7c64f60b
   az provider register --namespace Microsoft.ContainerRegistry --subscription 56397b98-2390-4ae1-85d3-463f7c64f60b
   az provider register --namespace Microsoft.KeyVault --subscription 56397b98-2390-4ae1-85d3-463f7c64f60b
   ```

2. *(Nice to have)* **Grant me `Contributor`** on the sub. I'll only create things inside `niftyexp` and tagged as above, so blast radius is limited. Portal path: Subscriptions → `Azure subscription 1` → Access control (IAM) → Add role assignment → Contributor → `chishty@niftyitsolution.com`.

3. *(Optional)* If you'd rather keep sub-level access tight, just grant me **`Contributor`** on the `niftyexp` RG (I only have Owner there today, which is fine, but Contributor + provider-register combined is the cleanest setup). Actually, Owner already covers Contributor's permissions inside the RG, so this is a no-op — skip if you did #2.

If neither #1 nor #2 is possible this weekend, no problem — I'll do the K8s labs on a local Kubernetes cluster (`kind`) and use GitHub Container Registry, and swing back to real AKS/ACR/KV once you have a minute.

Thanks!
