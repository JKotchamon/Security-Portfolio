# Chapter 1 — The Blueprint | Operation Dead Deploy

**Learning track:** Mad Hat Labs — Azure Fundamentals  
**Focus:** Cloud infrastructure, governance, deployment forensics, and security investigation  
**Environment:** Live multi-user Azure training tenant  
**Access:** Reader (read-only)  
**Status:** Draft — add your own observations and redacted screenshots before publishing

> **Scope and integrity:** This is a training-environment investigation, not production experience. No challenge flags, answer strings, tenant/subscription IDs, usernames, or other sensitive identifiers are included.

## Overview

Chapter 1 introduced the fundamentals of Microsoft Azure and the governance controls that help organisations manage cloud resources. I used these concepts as the foundation for **Operation Dead Deploy**, a scenario-based investigation into a non-compliant Azure deployment and the policy configuration that allowed it.

This case study separates **what I learned** from **what I observed during the investigation**.

## Part 1 — Chapter 1: The Blueprint

### 1.1 What is Microsoft Azure?

Microsoft Azure is a cloud computing platform offering on-demand computing, storage, networking, and application services. Instead of purchasing and maintaining all physical infrastructure on-premises, organisations can provision resources as needed and pay for their usage.

Key concepts:

- **On-premises vs cloud:** Own and maintain servers versus consume cloud services.
- **CapEx vs OpEx:** Upfront hardware investment versus ongoing operational spending.
- **Scalability:** Scale up by adding capacity to a resource or scale out by adding resources.
- **Service models:** **IaaS** (manage guest OS and applications), **PaaS** (focus on applications while the provider manages the platform), and **SaaS** (consume a managed application).
- **Cost awareness:** Resources and dependencies may continue generating charges; monitor spending and remove unneeded resources.

**Security relevance:** Cloud security requires understanding which responsibilities remain with the customer, even when Microsoft manages the underlying infrastructure.

### 1.2 Azure resource hierarchy

Azure resources are organised under a tenant using the following resource hierarchy:

```text
Microsoft Entra tenant (identity boundary)
└── Management groups
    └── Subscriptions
        └── Resource groups
            └── Resources
```

- **Management groups:** Organise subscriptions and support governance at scale.
- **Subscriptions:** Define important billing and access scopes.
- **Resource groups:** Logical containers for resources and a practical scope for investigations.
- **Resources:** Individual Azure services such as virtual machines, storage accounts, and networks.

Role-Based Access Control (**RBAC**) and Azure Policy assignments can apply at higher scopes and affect child scopes. The **principle of least privilege** reduces the impact, or *blast radius*, of a compromised account.

I also learned the distinction between **Object IDs**, which identify Entra identities, and **Resource IDs**, which identify Azure resources.

**Security relevance:** The scope of a role assignment determines how much of the environment an identity may access.

### 1.3 Tags, naming, and cost management

Tags are key-value metadata used to identify resources by owner, environment, application, cost centre, or lifecycle. They help analysts understand what a resource is for and who should be contacted about it.

Consistent resource names improve discoverability. A common naming pattern is:

```text
{resource-type}-{workload}-{environment}-{region}-{instance}
```

For example, a resource group name might follow `rg-webapp-prod-eastus-001`. Some resource types, such as storage accounts, have additional naming restrictions.

Cost Management and budgets help detect unexpected spending. **Budget alerts notify; they do not automatically stop resources.**

**Security relevance:** Unexpected resources, missing tags, naming outliers, and cost spikes can be useful investigation leads, but they are not proof of compromise by themselves.

### 1.4 Infrastructure as Code: ARM and Bicep

Infrastructure as Code (**IaC**) defines infrastructure in files rather than relying entirely on manual portal actions. This supports repeatable deployments, change review, and version control.

- **ARM templates:** JSON-based Azure Resource Manager deployment definitions.
- **Bicep:** A more concise Azure-focused language compiled into ARM templates.
- **Terraform:** A separate multi-cloud IaC tool.

Important commands covered in the lesson:

```bash
bicep build main.bicep
az deployment group what-if --resource-group <resource-group> --template-file main.bicep
az deployment group create --resource-group <resource-group> --template-file main.bicep
```

**Security relevance:** Deployment history and related logs can help reconstruct changes and identify unexpected provisioning activity.

### 1.5 Azure Policy and Resource Locks

**Azure Policy** evaluates resources against governance rules. A **policy definition** describes the rule, a **policy assignment** applies it to a scope, and an **initiative** groups definitions.

| Policy effect | Purpose |
|---|---|
| `deny` | Blocks non-compliant requests |
| `audit` | Allows requests but reports non-compliance |
| `append` | Adds specified properties |
| `modify` | Changes specified properties |
| `deployIfNotExists` | Can deploy related resources when conditions require it |
| `auditIfNotExists` | Reports when a required related resource is absent |
| `disabled` | Disables policy evaluation |

**Resource Locks** provide a different kind of protection:

- **CanNotDelete:** Prevents deletion while allowing permitted modifications.
- **ReadOnly:** Prevents modification and deletion.

**Security relevance:** A policy that only audits violations provides visibility, not preventive enforcement. Resource Locks protect against certain destructive or modifying operations.

---

## Part 2 — Operation Dead Deploy: Investigation

### Scenario

An intern with temporary Contributor access deployed a test environment that did not follow the organisation's resource naming convention. Acting as an on-call investigator with Reader access, my task was to locate the anomalous resources, inspect their metadata, review the deployment trail, and determine why the naming policy did not block the deployment.

### Environment

| Item | Details |
|---|---|
| Platform | Microsoft Azure |
| Lab | Mad Hat Labs — Operation Dead Deploy |
| Environment type | Live multi-user Azure training tenant |
| Access level | Reader |
| Interfaces | Azure Portal |
| Areas investigated | Resource Groups, Tags, Deployments, Azure Policy |

### Investigation

#### Stage 1 — Locate the naming anomaly

**Question:** Which resource group does not follow the expected naming convention?

1. Opened **Resource Groups** in the Azure Portal.
2. Compared the visible resource group names against the naming pattern introduced in Chapter 1.
3. **My observation:** [Describe the naming anomaly in your own words without publishing challenge answers or sensitive identifiers.]
4. **My conclusion:** [Explain why the anomaly warranted further review.]

**Evidence:** `screenshots/01-resource-groups-redacted.png`

#### Stage 2 — Inspect the resource and tags

**Question:** What does the resource metadata reveal about its purpose, ownership, or lifecycle?

1. Opened the relevant resource group and inspected its resources.
2. Reviewed the **Tags** blade and considered which metadata could establish context.
3. **My observation:** [Describe the types of tags or missing metadata; do not disclose any challenge flag values.]
4. **My conclusion:** [Explain what the metadata did and did not establish.]

**Evidence:** `screenshots/02-tags-redacted.png`

#### Stage 3 — Trace the deployment

**Question:** What deployment activity explains how the resources were provisioned?

1. Opened the resource group's **Deployments** blade.
2. Reviewed available deployment information, including timing, status, and associated details.
3. **My observation:** [Describe what the deployment record showed without disclosing its identifying name or challenge answers.]
4. **My conclusion:** [Explain how the evidence contributed to a timeline; do not assert an actor unless the evidence supports it.]

**Evidence:** `screenshots/03-deployments-redacted.png`

#### Stage 4 — Identify the governance gap

**Question:** Why was a non-compliant resource allowed even though a naming policy was assigned?

1. Opened the resource group's **Policies** blade.
2. Reviewed the naming policy's compliance state and assignment.
3. Examined the assignment's **Parameters** tab, focusing on the **Effect** parameter.
4. The scenario identifies the effect as **`audit`**, which records non-compliance without preventing creation.
5. Compared this with **`deny`**, which blocks requests that violate an applicable policy.

**Finding:** The naming policy's `audit` configuration explains why the deployment could proceed despite a reported naming violation. This is a governance enforcement gap **if** the organisation intended the policy to prevent non-compliant deployments.

**Evidence:** `screenshots/04-policy-effect-redacted.png`

### What broke / what surprised me

> Replace this section with an authentic experience from your lab. Do not present an invented obstacle as something you encountered.

- **Challenge or unexpected result:** [What confused you or took extra time?]
- **How I investigated it:** [What did you check next?]
- **What changed in my understanding:** [What did you learn from the difficulty?]

### Findings and recommendations

| Finding | Security significance | Recommendation |
|---|---|---|
| Naming policy effect was `audit` | Violations could be reported without blocking deployment | Confirm the intended control objective; use `deny` if preventive enforcement is required, after testing impact |
| Temporary Contributor access was part of the scenario | Write permissions increase the scope of potential changes | Review access scope, approval, duration, and timely removal |
| Resource metadata supports investigation | Weak or inconsistent tagging makes ownership and cost attribution harder | Require and validate owner, environment, and lifecycle tags where appropriate |

**Assessment:** A non-compliant resource is a governance concern, but the naming violation alone does not establish malicious activity or a compromised identity.

### What I learned

- Azure resource hierarchy helps me identify where resources live and where governance controls apply.
- Tags and deployment records provide useful context when investigating unexpected cloud resources.
- The difference between **`audit`** and **`deny`** is critical: detection and prevention are not the same.
- Reader access can support meaningful investigation without granting permission to modify resources.
- **What I would do differently next time:** [Add your own specific improvement, such as preparing an evidence checklist or recording timestamps as you investigate.]

### Conclusion

Operation Dead Deploy demonstrates how a deployment can be flagged as non-compliant yet still be permitted by its assigned policy. The central lesson is that the existence of a security control does not guarantee enforcement: its configuration and scope must match the organisation's security objectives.

## Evidence checklist

Add **6–12 meaningful, redacted screenshots** from your own session, for example:

- [ ] Resource group list showing a naming outlier
- [ ] Resource group overview
- [ ] Resource list within the group
- [ ] Tags blade with sensitive values blurred
- [ ] Deployment history with identifying names blurred
- [ ] Relevant deployment detail view
- [ ] Policy compliance view
- [ ] Policy assignment view
- [ ] Policy Parameters tab showing `audit`

Use only screenshots you actually captured. Remove unused entries, and add links or Markdown images once the files are uploaded.

## Publication checklist

- [ ] All investigation steps reflect actions I actually performed.
- [ ] All placeholders have been replaced with my own observations.
- [ ] No flags, answer strings, GUIDs, usernames, or tenant/subscription IDs are exposed.
- [ ] Screenshots are redacted and correctly linked.
- [ ] Training environment and Reader access are described accurately.
- [ ] The report is committed to my GitHub portfolio.

---

*Author's note: This write-up is based on an authorised educational lab and documents methodology and learning rather than disclosing course solutions.*
