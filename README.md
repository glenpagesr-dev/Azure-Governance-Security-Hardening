# 🔐 Azure Governance & Security Hardening Lab
 
> **Lab 05 | Difficulty:** Beginner &nbsp;|&nbsp; **Estimated Time:** 45 Minutes &nbsp;|&nbsp; **Cloud:** Microsoft Azure
 
---
 
## 📺 [Watch Me Build the Lab Here](https://www.loom.com/share/c2693a043e8e4a629ba5e99f6e6e71b0)
 
---
 
## 🎯 Objective
 
In this lab, I step into the role of a **Cloud Administrator** responsible for securing and governing an Azure environment. Rather than operating as a privileged Owner who can do anything, the focus is on restricting access for others, enforcing policy guardrails, and setting up cost visibility — three pillars of real-world cloud governance.
 
**By the end of this lab, you will have:**
- Created a restricted user identity in **Microsoft Entra ID**
- Assigned **RBAC Reader** permissions scoped to a single resource group
- Enforced an **Azure Policy** to block expensive VM sizes
- Configured a **Cost Management Budget** with spending alerts
---
 
## 🏗️ Architecture Diagram
 
```
┌─────────────────────────────────────────────────────────────────────────┐
│                        AZURE SUBSCRIPTION                               │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │              rg-lab05-gov-[yourname]  (Resource Group)           │  │
│  │                                                                  │  │
│  │   ┌─────────────┐    RBAC (Reader)    ┌──────────────────────┐  │  │
│  │   │  👤 Admin    │ ─────────────────► │  👤 Junior Developer  │  │  │
│  │   │  (You)       │                   │  junior-dev-[yourname] │  │  │
│  │   │  Full Access │                   │  View Only — No Write  │  │  │
│  │   └──────┬───────┘                   └──────────┬─────────────┘  │  │
│  │          │                                      │                 │  │
│  │          │ Applies Policy                       │ Tries to create │  │
│  │          ▼                                      ▼                 │  │
│  │   ┌─────────────────────┐            ┌──────────────────────┐   │  │
│  │   │  🛡️ Azure Policy     │            │  🚫 ACCESS DENIED    │   │  │
│  │   │  Restrict-VM-Sizes  │            │  Authorization Failed │   │  │
│  │   │  Allowed: B1s, B1ms │            └──────────────────────┘   │  │
│  │   └─────────────────────┘                                        │  │
│  │          │                                                        │  │
│  │          │ Blocks non-compliant deploy                            │  │
│  │          ▼                                                        │  │
│  │   ┌──────────────────────────────────────────────────────────┐   │  │
│  │   │  🖥️ VM Deploy Attempt: Standard_D2s_v3                   │   │  │
│  │   │  ❌  Policy Check Failed → "Restrict-VM-Sizes"           │   │  │
│  │   └──────────────────────────────────────────────────────────┘   │  │
│  │                                                                  │  │
│  │   ┌──────────────────────────────────────────────────────────┐   │  │
│  │   │  💰 Cost Management Budget                               │   │  │
│  │   │  Name: Monthly-Lab-Budget  |  Limit: $50/mo              │   │  │
│  │   │  Alert: 80% threshold → Email notification               │   │  │
│  │   └──────────────────────────────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  🆔 Microsoft Entra ID (formerly Azure Active Directory)         │  │
│  │  └── Users: junior-dev-[yourname]@[tenant].onmicrosoft.com      │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```
 
---
 
## ✅ Prerequisites
 
- [ ] Active Azure Subscription
- [ ] Completed Week 5 Video Modules
- [ ] Ability to open an **Incognito / Private browser window**
---
 
## 📋 Lab Variables (Naming Convention)
 
| Variable | Value |
|---|---|
| Resource Group | `rg-lab05-gov-[yourname]` |
| Test User | `junior-dev-[yourname]@[yourtenant].onmicrosoft.com` |
| Display Name | `Junior Developer` |
| Policy Name | `Restrict-VM-Sizes` |
| Budget Name | `Monthly-Lab-Budget` |
| Allowed VM SKUs | `Standard_B1s`, `Standard_B1ms` |
| Budget Amount | `$50 / billing month` |
| Alert Threshold | `80%` |
 
---
 
## 🚀 Step-by-Step Instructions
 
### Phase 1 — Set Up the Playground
 
1. Log in to the [Azure Portal](https://portal.azure.com) as your main admin account.
2. Search for **Resource Groups** → click **+ Create**.
3. Configure the resource group:
   - **Name:** `rg-lab05-gov-[yourname]`
   - **Region:** `East US`
4. Click **Review + create** → **Create**.
---
 
### Phase 2 — RBAC (Role-Based Access Control)
 
> **Goal:** Simulate hiring a Junior Developer who can only *view* resources — not create or delete them.
 
1. Search for **Microsoft Entra ID** in the top search bar.
2. In the left menu, click **Users** → **New user** → **Create new user**.
3. Fill in the **Basics** tab:
   - **User principal name:** `junior-dev-[yourname]`
   - **Display name:** `Junior Developer`
   - **Password:** Uncheck *Auto-generate* → enter a password you'll remember
4. Click **Review + create** → **Create**.
**Assign the Reader Role:**
 
5. Navigate to **Resource Groups** → `rg-lab05-gov-[yourname]`.
6. Click **Access control (IAM)** → **+ Add** → **Add role assignment**.
7. **Role tab:** Search for `Reader` → select it → click **Next**.
8. **Members tab:** Click **+ Select members** → search for `Junior Developer` → select → **Next**.
9. Click **Review + assign** → **Review + assign**.
---
 
### Phase 3 — Verify Access (The "Access Denied" Test)
 
> **Goal:** Prove the Reader role restriction actually works.
 
1. Open a **New Incognito / Private browser window**.
2. Go to [portal.azure.com](https://portal.azure.com).
3. Log in as `junior-dev-[yourname]@[yourdomain]`.
4. Navigate to **Resource Groups** → `rg-lab05-gov-[yourname]`.
5. Click **+ Create** inside the group → search for **Storage Account** → try to create one.
**Expected Result:**
> ❌ Validation fails OR the Create button is greyed out. You may see a red banner: *"You do not have authorization to perform this action."*
 
6. ✅ **Success!** Close the Incognito window and return to your Admin account.
---
 
### Phase 4 — Azure Policy (Preventing Expensive Mistakes)
 
> **Goal:** Enforce a guardrail so no one — not even you — can deploy oversized VMs in this resource group.
 
1. Search for **Policy** in the main Azure search bar.
2. Under **Authoring**, click **Assignments** → **+ Assign policy**.
3. Fill in the **Basics** tab:
   - **Scope:** Click `...` → Select your subscription → select `rg-lab05-gov-[yourname]` as the Resource Group → **Select**
   - **Policy definition:** Click `...` → search `Allowed virtual machine size SKUs` → select it
   - **Assignment name:** `Restrict-VM-Sizes`
4. Click **Next** to reach **Parameters**:
   - Uncheck *"Only show parameters that need input"*
   - **Allowed Size SKUs:** Select `Standard_B1s` and `Standard_B1ms` only
5. Click **Review + create** → **Create**.
> ⏱️ **Note:** Policies can take **15–30 minutes** to fully replicate across Azure. If the test below doesn't fail immediately, wait and retry.
 
---
 
### Phase 5 — Testing the Policy
 
1. Go to `rg-lab05-gov-[yourname]` → **+ Create** → **Virtual Machine**.
2. Fill in the **Basics** tab:
   - **Name:** `vm-policy-test`
   - **Image:** `Ubuntu Server`
   - **Size:** Change to `Standard_D2s_v3` (any non-B1 size)
3. Click **Review + create**.
**Expected Result:**
> ❌ **Validation Failed** — Click the error. It should state *"Policy check failed"* and list `Restrict-VM-Sizes` as the reason.
 
4. ✅ **Success!** The governance guardrail is working.
---
 
### Phase 6 — Cost Management (Budgets)
 
> **Goal:** Set a spending limit so you get notified before costs run away.
 
1. Go to `rg-lab05-gov-[yourname]`.
2. In the left menu, search for **Budgets** (under Cost Management) → click **+ Add**.
3. Fill in the **Create budget** form:
   - **Name:** `Monthly-Lab-Budget`
   - **Reset period:** `Billing month`
   - **Creation date:** Today
   - **Expiration date:** 1 year from today
   - **Amount:** `50` (USD)
4. Click **Next**.
5. Set the alert:
   - **Type:** `Actual`
   - **% of budget:** `80`
   - **Alert recipients (email):** Enter your personal email address
6. Click **Create**.
---
 
## 🔧 Troubleshooting
 
| Issue | Root Cause | Fix |
|---|---|---|
| Junior Dev can still create resources | Role was assigned at the wrong scope | Confirm the **Reader** role is assigned to the specific Resource Group — not the Subscription |
| Policy didn't block the VM | Policy replication lag | Wait **15–30 minutes** and try the validation test again |
| Can't find Budgets in the portal | Cost Management may be at subscription level | Try navigating to **Cost Management + Billing** from the main search bar |
 
---
 
## 🧹 Clean Up Resources
 
> ⚠️ **Important:** Complete both steps to avoid orphaned identities and unexpected charges.
 
1. **Delete the Resource Group:**
   - Go to **Resource Groups** → `rg-lab05-gov-[yourname]` → **Delete resource group**
   - Type the name to confirm → **Delete**
2. **Delete the Test User:**
   - Go to **Microsoft Entra ID** → **Users**
   - Find `junior-dev-[yourname]` → **Delete user** → confirm
---
 
## 💡 Key Concepts Covered
 
| Concept | What It Does |
|---|---|
| **Microsoft Entra ID** | Identity provider — manages users, groups, and authentication |
| **RBAC (Role-Based Access Control)** | Controls *who* can do *what* on *which* Azure resources |
| **Reader Role** | View-only access — no ability to create, modify, or delete |
| **Azure Policy** | Enforces compliance rules before resources are deployed |
| **Allowed VM Size SKUs** | Policy definition that restricts which VM types can be created |
| **Cost Management Budget** | Proactive spending guardrail with threshold-based email alerts |
 
---
 
## 🛠️ Tools & Services Used
 
![Azure](https://img.shields.io/badge/Microsoft_Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Entra ID](https://img.shields.io/badge/Microsoft_Entra_ID-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Azure Policy](https://img.shields.io/badge/Azure_Policy-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Cost Management](https://img.shields.io/badge/Cost_Management-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
 
---
 
## 👤 Author
 
**Glen Page** — Cloud Engineer  
📎 [GitHub](https://github.com/glenpagesr-dev) | 🔗 [LinkedIn](https://linkedin.com/in/glen-page-862730246)
 
---
 
*Part of my Azure Cloud Engineering portfolio. Built to demonstrate real-world governance patterns aligned to enterprise security and cost control best practices.*
 
