# Milestone 4 — Identity, Access and Secrets

*Phase 3 · ~2.5 weeks · Prerequisite: Milestone 3*

> Placed before IaC and containers on purpose. Every resource you create from here needs an identity decision, and retrofitting security is the habit this JD's "solutions meet security standards" line exists to filter out.

**Learning topics**
* Entra ID objects: users, groups, service principals, app registrations, managed identities (system- vs user-assigned)
* Azure RBAC: role definitions, assignments, scope inheritance, deny assignments, propagation delay
* Least privilege as a *practice*: start from zero, add what fails
* **OIDC federated credentials** — why a long-lived credential in CI is the highest-risk thing in most pipelines, and how to have none
* Workload identity: how a pod's service account becomes an Azure identity
* Key Vault: secrets vs keys vs certificates, RBAC vs access policies, soft delete, purge protection, rotation, versioning
* OAuth2/OIDC flows: authorization code with PKCE for the SPA, client credentials for service-to-service

**Build**
A custom role definition granting exactly what `cargo-api` needs, derived empirically — assign nothing, run, read the error, add one action, repeat. `docs/security/iam-model.md` recording who can do what, where, and why.

**Cloud resources**
Entra groups `platform-admins`, `workload-developers`, `readers` with scoped role assignments · Key Vault with RBAC, soft delete and purge protection · a user-assigned managed identity for the VM · a federated credential for GitHub Actions.

**DevOps work**
Replace any service principal secret with OIDC federation. The pipeline authenticates to Azure with **no secret in the repository**. Then break the subject claim condition deliberately and read the failure.

**Operations exercise**
**The one you must do:** delete a role assignment the pipeline depends on, then diagnose the failed deployment from the error text alone. Then set a Key Vault secret to expire mid-session and watch the application fail. Write both up as incident records.

**Security considerations**
Database credentials leave the environment file and live in Key Vault, read via managed identity. Deliberately over-assign `Owner` at subscription scope, write down the blast radius in one paragraph, then fix it.

**Expected outcome**
Zero stored credentials anywhere: not in Git, not in GitHub secrets, not on the VM. `docs/incidents/INC-001-broken-role-assignment.md` exists, written during the drill.

**Evidence of mastery**
Explain managed identity vs service principal vs OIDC federated credential and when each is right. Given "the pipeline cannot deploy", list the five IAM causes in check order. Explain what an attacker could do with your pipeline's identity — and be satisfied with the answer.
