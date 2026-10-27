
# 🔐 Chapter 1 – The Blueprint
## Microsoft Azure Fundamentals & Operation Dead Deploy

**Learning Platform:** Mad Hat Labs  
**Focus:** Azure Fundamentals, Cloud Security & Governance  
**Environment:** Live multi-user Azure training tenant  
**Access Level:** Reader (Read-Only)  
**Status:** In Progress – Evidence and observations to be added

---

# 📚 Part 1 – Azure Fundamentals

## 1.1 What is Microsoft Azure?

Microsoft Azure is a cloud computing platform that allows businesses to rent computing resources instead of buying and maintaining physical servers.

Azure provides services such as:
- Virtual Machines (VMs)
- Cloud Storage
- Virtual Networks
- Databases
- Security and Monitoring

### On-Premises vs Cloud Computing

| On-Premises | Microsoft Azure |
|---|---|
| Buy physical servers | Rent cloud resources |
| High upfront cost (CapEx) | Operational spending (OpEx) |
| Maintain hardware yourself | Microsoft manages physical infrastructure |
| Scaling requires new hardware | Scale resources on demand |
| Limited to owned infrastructure | Global cloud infrastructure |

### Why Businesses Use Azure

1. **Cost Efficiency:** Pay for the resources you use.
2. **Scalability:** Increase or decrease resources based on demand.
3. **Global Reach:** Deploy applications in different regions.
4. **Security:** Use Microsoft's cloud security infrastructure.
5. **Flexibility:** Deploy and remove resources when needed.

### Three Cloud Service Models

| Model | Meaning | Example |
|---|---|---|
| IaaS | Infrastructure as a Service | Azure Virtual Machines |
| PaaS | Platform as a Service | Azure App Service |
| SaaS | Software as a Service | Microsoft 365 |

**Key Takeaway:** Cloud computing reduces the need to own physical infrastructure, but customers still have security responsibilities.

---

## 1.2 Azure Resource Hierarchy

Azure uses a hierarchy to organise resources, manage access, apply security policies, and track costs.

### Azure Hierarchy

```text
Microsoft Entra Tenant
│
└── Management Groups
    │
    └── Subscriptions
        │
        └── Resource Groups
            │
            └── Resources
```

### What Each Level Does

| Level | Purpose |
|---|---|
| Tenant | Organisation's identity boundary |
| Management Group | Organises subscriptions |
| Subscription | Billing and access scope |
| Resource Group | Logical container for resources |
| Resource | VM, storage account, network, etc. |

### Important Concepts

**Role-Based Access Control (RBAC)**

Controls who can access Azure resources and what actions they can perform.

**Principle of Least Privilege**

Users should only receive the permissions required to complete their work.

**Inheritance**

RBAC permissions and Azure Policy assignments can apply at higher scopes and affect resources below them.

**Blast Radius**

The amount of damage or access possible if an account is compromised.

### Azure Identifiers

| Identifier | Purpose |
|---|---|
| Tenant ID | Identifies an Entra tenant |
| Subscription ID | Identifies an Azure subscription |
| Object ID | Identifies a user or other Entra identity |
| Resource ID | Identifies a specific Azure resource |

### Important Azure CLI Commands

```bash
az account show
az account list
az group list
az resource list
```

### Important Notes

- Each resource belongs to one Resource Group.
- Resource Groups cannot be nested.
- Deleting a Resource Group deletes the resources inside it.
- Resource Group location is where its metadata is stored.
- Costs roll up through subscriptions and management groups.

**Key Takeaway:** Understanding Azure hierarchy is important for access control, security investigations, and reducing the blast radius of compromised accounts.

---

## 1.3 Tags, Naming & Cost Management

### A. Azure Tags

Tags are key-value pairs used to identify and organise Azure resources.

Example:

| Tag | Value |
|---|---|
| Environment | Production |
| Owner | SecurityTeam |
| CostCenter | IT-001 |
| Application | WebApp |
| ExpiresOn | 2026-12-31 |

### Why Tags Matter

- Identify resource owners.
- Organise resources by environment.
- Track cloud spending.
- Support automation.
- Help investigate suspicious resources.

**Important:** Tags do not automatically inherit from Resource Groups to resources unless configured.

### B. Azure Naming Conventions

Consistent naming helps teams identify resources quickly.

**Recommended Pattern:**

```text
{resource-type}-{workload}-{environment}-{region}-{instance}
```

**Example:**

```text
rg-webapp-prod-eastus-001
vm-webapp-prod-eastus-001
vnet-webapp-prod-eastus-001
```

### Common Resource Prefixes

| Prefix | Resource |
|---|---|
| rg | Resource Group |
| vm | Virtual Machine |
| vnet | Virtual Network |
| nsg | Network Security Group |
| pip | Public IP Address |
| st | Storage Account |
| kv | Key Vault |
| app | Web App |
| func | Function App |
| sql | SQL Server |
| aks | Kubernetes Cluster |
| cr | Container Registry |
| log | Log Analytics Workspace |

**Storage Account Naming Rules:**
- Lowercase letters and numbers only.
- No hyphens or underscores.
- Between 3 and 24 characters.
- Globally unique name.

### C. Azure Cost Management

Azure resources can generate charges even when they are not actively used.

Important cost factors:
- Running Virtual Machines
- Storage resources
- Public IP addresses
- Data transferred out of Azure
- Different regional pricing

### Azure Budgets

Budgets help monitor cloud spending.

Example:

```text
Monthly Budget: $100

50% = $50 Alert
80% = $80 Alert
100% = $100 Alert
```

**Important:** Budget alerts notify users but do not automatically stop resources.

**Security Connection:** Unexpected spending may indicate unauthorised resources, misconfigurations, or cryptojacking.

---

## 1.4 Infrastructure as Code (IaC): ARM & Bicep

Infrastructure as Code (IaC) allows cloud infrastructure to be created and managed using code rather than manual configuration.

### Benefits of IaC

1. **Declarative:** Describe the desired infrastructure.
2. **Idempotent:** Repeated deployments should produce the same desired result.
3. **Version-Controlled:** Track changes using Git.

### ARM vs Bicep vs Terraform

| Tool | Description |
|---|---|
| ARM Templates | Azure infrastructure templates written in JSON |
| Bicep | Simpler Azure-focused infrastructure language |
| Terraform | Multi-cloud IaC tool using HCL |

### ARM Template

ARM stands for Azure Resource Manager.

ARM templates use JSON and commonly contain:

- `$schema`
- `contentVersion`
- `parameters`
- `variables`
- `resources`
- `outputs`

### Bicep Example

```bicep
resource demoStorage 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'stmadhatdemo001'
  location: 'eastus'
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

Bicep is easier to read than ARM JSON and is compiled into an ARM template during deployment.

### Important Commands

**Convert Bicep to ARM JSON**

```bash
bicep build main.bicep
```

**Preview Deployment Changes**

```bash
az deployment group what-if \
  --resource-group <resource-group> \
  --template-file main.bicep
```

**Deploy Resources**

```bash
az deployment group create \
  --resource-group <resource-group> \
  --template-file main.bicep
```

### Why IaC Matters for Cybersecurity

- Tracks infrastructure changes.
- Supports deployment auditing.
- Helps identify unexpected resources.
- Improves consistency between environments.
- Supports investigation of suspicious deployments.

**Key Takeaway:** Infrastructure as Code makes cloud deployments more consistent, repeatable, and easier to investigate.

---

## 1.5 Azure Policy & Resource Locks

Azure Policy and Resource Locks are security controls used to manage and protect cloud resources.

### A. Azure Policy

Azure Policy evaluates resources against organisational rules.

### Three Important Concepts

| Concept | Description |
|---|---|
| Policy Definition | Defines the rule |
| Policy Assignment | Applies the rule to a scope |
| Initiative | Groups multiple policies |

### Seven Common Azure Policy Effects

| Effect | Purpose |
|---|---|
| `deny` | Blocks non-compliant requests |
| `audit` | Allows requests but reports violations |
| `append` | Adds specified properties |
| `modify` | Changes specified properties |
| `deployIfNotExists` | Deploys related resources when conditions are met |
| `auditIfNotExists` | Reports when a required related resource is missing |
| `disabled` | Disables policy evaluation |

### Audit vs Deny

**Audit**
- Allows deployment.
- Reports non-compliance.
- Does not prevent the resource from being created.

**Deny**
- Blocks non-compliant requests.
- Prevents the resource from being created.

### Common Azure Policy Examples

- Restrict resource locations.
- Restrict VM sizes.
- Require encryption.
- Require specific resource tags.
- Restrict public IP addresses.
- Disable public network access to storage accounts.

### B. Azure Resource Locks

Resource Locks protect important resources from unwanted changes or deletion.

| Lock Type | Behaviour |
|---|---|
| CanNotDelete | Prevents deletion |
| ReadOnly | Prevents modification and deletion |

### Important Notes

- Locks can be applied at subscription, Resource Group, or resource level.
- Locks can inherit downward.
- Locks can block operations even when a user has the Owner role.
- An authorised user must remove the lock before performing a blocked operation.

**Key Takeaway:** Azure Policy controls compliance, while Resource Locks protect resources from accidental modification or deletion.

---

# 🔎 Part 2 – Operation Dead Deploy

## 2.1 Scenario

An intern with temporary Contributor access deployed a test environment that did not follow the organisation's naming conventions.

My task was to investigate the resources, review deployment activity, and determine why Azure Policy allowed the non-compliant deployment.

The investigation was performed in a live multi-user Azure training tenant using Reader access.

## 2.2 Environment

| Component | Details |
|---|---|
| Platform | Microsoft Azure |
| Lab | Mad Hat Labs – Operation Dead Deploy |
| Environment | Live multi-user Azure training tenant |
| Access | Reader (Read-Only) |
| Interface | Azure Portal |
| Services | Resource Groups, Tags, Deployments, Azure Policy |

## 2.3 Investigation

### Stage 1 – Identify the Resource Group

**Objective:** Find the Resource Group that does not follow the naming convention.

**Investigation Steps:**

1. Navigate to Azure Portal.
2. Open Resource Groups.
3. Review the Resource Group names.
4. Identify the naming anomaly.

**My Observation:**

> [Describe what you found without revealing challenge answers.]

**Evidence:**

![Resource Group Investigation](screenshots/01-resource-groups.png)

---

### Stage 2 – Inspect Resource Tags

**Objective:** Understand the resource's ownership and purpose.

**Investigation Steps:**

1. Open the identified Resource Group.
2. Review the resources inside it.
3. Navigate to Tags.
4. Inspect the available metadata.

**My Observation:**

> [Describe the relevant tags and what they helped you understand. Do not reveal flags.]

**Evidence:**

![Resource Tags](screenshots/02-resource-tags.png)

---

### Stage 3 – Investigate Deployment History

**Objective:** Understand how and when the resources were deployed.

**Investigation Steps:**

1. Open the Resource Group.
2. Navigate to Deployments.
3. Review deployment history.
4. Examine available deployment details.

**My Observation:**

> [Explain what the deployment records revealed without exposing identifying names.]

**Evidence:**

![Deployment History](screenshots/03-deployments.png)

---

### Stage 4 – Investigate Azure Policy

**Objective:** Determine why the naming policy did not prevent the deployment.

**Investigation Steps:**

1. Open the Resource Group's Policies blade.
2. Review the naming policy's compliance status.
3. Open Policy Assignments.
4. Inspect the Parameters tab.
5. Review the Effect parameter.

**Finding:**

The naming policy was configured with `audit`.

This allowed non-compliant resources to be created while reporting them as policy violations.

The `deny` effect would block requests that violate an applicable naming policy.

**Evidence:**

![Azure Policy Effect](screenshots/04-policy-effect.png)

---

## 2.4 What Broke / What Surprised Me

**Challenge:**

> [Describe a real problem or unexpected result you experienced.]

**How I Investigated It:**

> [Explain what you checked to resolve the problem.]

**What I Learned:**

> [Explain what you would do differently next time.]

---

## 2.5 Findings & Recommendations

| Finding | Recommendation |
|---|---|
| Naming policy configured with Audit | Consider using Deny if enforcement is required |
| Temporary Contributor access | Review permissions and access duration |
| Resource ownership needs clear identification | Require appropriate resource tags |
| Non-compliant resources were permitted | Regularly review Azure Policy compliance |

### Security Recommendations

1. **Enforce Naming Conventions:** Use Deny where appropriate after reviewing deployment impact.
2. **Apply Least Privilege:** Limit Contributor access to the minimum required scope and duration.
3. **Require Resource Tags:** Use tags to identify ownership, environment, and lifecycle.
4. **Monitor Compliance:** Regularly review Azure Policy compliance reports.

---

## 2.6 What I Learned

- How Azure organises resources using its hierarchy.
- How RBAC and least privilege affect cloud security.
- How resource tags and naming conventions support investigations.
- How to examine Azure deployment history.
- How ARM and Bicep support repeatable deployments.
- How Azure Policy evaluates resource compliance.
- The difference between `audit` and `deny`.
- Why Resource Locks are important for protecting critical resources.

**What I Would Improve Next Time:**

> [Add one specific improvement based on your investigation.]

---

## 2.7 Conclusion

Operation Dead Deploy demonstrated how a non-compliant Azure deployment could succeed even when a naming policy was assigned.

The key governance issue was the use of the `audit` effect rather than preventive enforcement.

This investigation helped me understand the importance of cloud governance, access control, deployment auditing, and security policy configuration.

**Key Lesson:** Having a security policy does not guarantee enforcement. The policy must be correctly configured to meet the organisation's security requirements.

---

# 📸 Evidence Checklist

Before publishing, add 6–12 meaningful screenshots from your own investigation.

- [ ] Resource Group list
- [ ] Resource Group overview
- [ ] Resources inside the Resource Group
- [ ] Resource Tags
- [ ] Deployment history
- [ ] Deployment details
- [ ] Policy compliance
- [ ] Policy assignment
- [ ] Policy Parameters showing Audit

**Security Reminder:** Redact tenant IDs, subscription IDs, usernames, deployment identifiers, challenge flags, and other sensitive information.

Never publish lab answers or flag values.

---

# 📌 Final Reflection

This chapter introduced the fundamentals of Microsoft Azure and demonstrated how cloud governance failures can be investigated.

The main skills covered were:

- Cloud infrastructure understanding
- Resource organisation
- Identity and access control
- Cloud cost awareness
- Infrastructure as Code
- Azure governance and compliance
- Deployment investigation
- Security reporting

These skills form a foundation for future investigations involving Microsoft Entra ID, Azure RBAC, monitoring, incident response, and Microsoft Sentinel.

---

## Disclaimer

This project was completed as part of an authorised Mad Hat Labs training environment.

It documents learning and investigation methodology rather than production security experience. Sensitive information and challenge answers must not be published.
