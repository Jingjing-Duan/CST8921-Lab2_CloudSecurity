# CST8921 – Cloud Industry Trends
## Lab 2 – Cloud Security Trends

---

## Introduction

In this lab, students will explore and understand the criticality of cloud security in industries. A crucial component of cloud security is focused on protecting data and business content, such as customer orders, secret design documents, and financial records. Preventing leaks and data theft is critical for maintaining your customers' trust and protecting the assets that contribute to your competitive advantage. Cloud security's ability to guard your data and assets makes it crucial to any company switching to the cloud.

Cloud security ensures your data and applications are readily available to authorized users. You'll always have a reliable method to access your cloud applications and information, helping you quickly act on any potential security issues.

---

## Objective

The goal of this lab activity is to familiarize students with the concepts, techniques, and use cases of cloud security using **AWS / Azure / GCP**.

---

## Prerequisites

- Basic understanding of cloud security concepts
- A computer with internet access
- Windows or Mac machine
- Web browser
- Cloud portal access with any cloud service provider (AWS, Azure, GCP)

---

## Lab Activity Overview

---

### Task 1: Create an Azure Policy – Allowed Locations

**Purpose:** Ensure resources can only be deployed in an approved region.

**Steps:**
1. Sign in to the **Azure Portal**
2. Search for **Policy**
3. Select **Definitions**
4. Search for the built-in policy: `Allowed locations`
5. Select the policy → Click **Assign**
6. Configure:
   - **Scope:** Subscription
   - **Allowed locations:** Canada Central
7. Click **Review + Create**

**Validation:**
- Attempt to create a resource in a different region (e.g., East US)
- Confirm deployment fails

---

### Task 2: Create a Virtual Network (Canada Central)

**Purpose:** Establish a secure network boundary.

**Steps:**
1. Search for **Virtual Networks**
2. Select **Create**
3. Configure:
   - **Region:** Canada Central
   - **Address space:** `10.0.0.0/16`
4. Do not add subnets yet
5. Click **Review + Create**

---

### Task 3: Create Subnets & Enable Storage Service Endpoint

**Purpose:** Ensure storage traffic stays within Azure's backbone network.

**Subnets to Create:**

| Subnet Name    | Address Range  | Purpose          |
|----------------|----------------|------------------|
| private-subnet | 10.0.1.0/24    | Secure access    |
| public-subnet  | 10.0.2.0/24    | Internet-facing  |

**Steps:**
1. Open the **Virtual Network**
2. Go to **Subnets**
3. Add `private-subnet`:
   - Enable **Service Endpoint**
   - Service: `Microsoft.Storage`
4. Add `public-subnet` (no service endpoint)

---

### Task 4: Create Network Security Group (NSG)

**Purpose:** Control inbound and outbound network traffic.

**Steps:**
1. Search for **Network Security Groups**
2. Create NSG in **Canada Central**
3. Associate NSG to `private-subnet`

---

### Task 5: Configure NSG Rules (Private Subnet)

**Outbound Rule – Allow Azure Storage**

| Setting       | Value           |
|---------------|-----------------|
| Destination   | Service Tag     |
| Service Tag   | Storage         |
| Action        | Allow           |
| Priority      | 100             |

**Outbound Rule – Deny Internet Access**

| Setting       | Value           |
|---------------|-----------------|
| Destination   | Internet        |
| Action        | Deny            |
| Priority      | 200             |

---

### Task 6: Configure NSG for Public Subnet (RDP Access)

**Inbound Rule – Allow RDP**

| Setting   | Value  |
|-----------|--------|
| Source    | Any    |
| Port      | 3389   |
| Protocol  | TCP    |

---

### Task 7: Create a Storage Account with File Share

**Purpose:** Secure storage access to only approved subnets.

**Steps:**
1. Create a **Storage Account**
   - Region: Canada Central
2. Go to **Networking**
3. Set:
   - **Network access:** Enabled from selected networks
   - Allow access only from `private-subnet`
4. Create an **Azure File Share**

---

### Task 8: Deploy Virtual Machines

Deploy two Windows VMs:

| VM Name    | Subnet         |
|------------|----------------|
| vm-private | private-subnet |
| vm-public  | public-subnet  |

- Enable **Azure Bastion**
- Use the **same credentials** for both VMs

> 📸 **Screenshot required:** VM overview pages

---

### Task 9: Test Storage Access from Private Subnet (Allowed)

1. Connect to `vm-private` using **Bastion**
2. Open **Windows PowerShell**
3. Run the following command:

```powershell
$key = @{
    String = "<storage-account-key>"
}
$acctKey = ConvertTo-SecureString @key -AsPlainText -Force

$cred = @{
    ArgumentList = "Azure\<storage-account-name>", $acctKey
}
$credential = New-Object System.Management.Automation.PSCredential @cred

$map = @{
    Name       = "Z"
    PSProvider = "FileSystem"
    Root       = "\\<storage-account-name>.file.core.windows.net\file-share"
    Credential = $credential
}
New-PSDrive @map
```

**Expected Result:** Azure file share successfully mapped to drive `Z:`

---

### Task 10: Test Storage Access from Public Subnet (Denied)

1. Connect to `vm-public` using Bastion
2. Repeat the same PowerShell command from Task 9

**Expected Result:** Access denied error

---

## Validation Summary

| Scenario                                  | Expected Outcome |
|-------------------------------------------|------------------|
| Resource creation outside Canada Central  | Blocked          |
| Storage access from private subnet        | Allowed          |
| Storage access from public subnet         | Denied           |

---

## Cleanup (Mandatory)

> ⚠️ To avoid unexpected costs, delete all resources after completing the lab.

Delete the following:
- Virtual Machines
- Storage Account
- Network Security Groups (NSGs)
- Virtual Network
- Azure Policy assignment

---

## Deliverables & Grading

### Lab Report Requirements
- Step-by-step explanation of each task
- Screenshots for each task
- Security analysis and observations
- Validation results
- Cleanup confirmation

### Submission
- Upload your report to **Brightspace → Assignments**

---

## Important Notes

> For grading, prepare a lab report with your findings and analysis and submit it via the **Assignments** tab in Brightspace.
