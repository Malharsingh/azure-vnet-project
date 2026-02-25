# 📋 Configuration Notes — Azure Virtual Network Project 1

## Project Overview
| Field | Value |
|-------|-------|
| Project Name | Basic Virtual Network Setup |
| Resource Group | rg-vnet-project-001 |
| Region | East US |
| Date Completed | February 2026 |
| Author | Malhar |

---

## 1. Virtual Network Configuration

| Field | Value |
|-------|-------|
| VNet Name | vnet-project-001 |
| Address Space | 10.0.0.0/16 |
| Region | East US |
| DNS Servers | Default (Azure-provided) |

### Subnets

| Subnet Name | Address Range | Purpose |
|-------------|--------------|---------|
| web-subnet | 10.0.1.0/24 | Public-facing web server tier |
| db-subnet | 10.0.2.0/24 | Internal database server tier |

**Why two subnets?**
Separating web and database tiers into different subnets is a security best practice. It allows granular NSG rules to be applied at each tier, ensuring the database is never directly exposed to the internet.

---

## 2. Network Security Group (NSG) Configuration

### NSG — nsg-web-subnet
**Associated to:** web-subnet

#### Inbound Rules

| Priority | Rule Name | Port | Protocol | Source | Destination | Action |
|----------|-----------|------|----------|--------|-------------|--------|
| 100 | Allow-SSH | 22 | TCP | Any | Any | Allow |
| 110 | Allow-HTTP | 80 | TCP | Any | Any | Allow |
| 65500 | DenyAllInBound | Any | Any | Any | Any | Deny |

**Why these rules?**
- Port 22 (SSH) — Required to remotely manage the Linux VM via terminal
- Port 80 (HTTP) — Required for web traffic to reach the web server
- Default Deny All — Azure automatically denies all other traffic not explicitly allowed

---

### NSG — nsg-db-subnet
**Associated to:** db-subnet

#### Inbound Rules

| Priority | Rule Name | Port | Protocol | Source | Destination | Action |
|----------|-----------|------|----------|--------|-------------|--------|
| 100 | Allow-From-WebSubnet | Any | Any | 10.0.1.0/24 | Any | Allow |
| 65500 | DenyAllInBound | Any | Any | Any | Any | Deny |

**Why these rules?**
- Only traffic originating from the web-subnet (10.0.1.0/24) is permitted
- All internet traffic is blocked — the DB server has no public IP
- This simulates a real-world secure database tier that only the application layer can access

---

## 3. Virtual Machine Configuration

### VM 1 — vm-web-01 (Web Server)

| Field | Value |
|-------|-------|
| VM Name | vm-web-01 |
| Operating System | Ubuntu Server 22.04 LTS |
| VM Size | Standard_B1s |
| vCPUs | 1 |
| RAM | 1 GB |
| Subnet | web-subnet |
| Private IP | 10.0.1.4 |
| Public IP | Yes (pip-vm-web-01) |
| Auto-Shutdown | Enabled |
| NSG | nsg-web-subnet |

---

### VM 2 — vm-db-01 (Database Server)

| Field | Value |
|-------|-------|
| VM Name | vm-db-01 |
| Operating System | Ubuntu Server 22.04 LTS |
| VM Size | Standard_B1s |
| vCPUs | 1 |
| RAM | 1 GB |
| Subnet | db-subnet |
| Private IP | 10.0.2.4 |
| Public IP | None (intentionally removed) |
| Auto-Shutdown | Enabled |
| NSG | nsg-db-subnet |

**Why no public IP on vm-db-01?**
A database server should never be directly accessible from the internet. Removing the public IP ensures it can only be reached through internal VNet routing, specifically from vm-web-01 via the private IP.

---

## 4. Connectivity Testing Results

### Test 1 — SSH into vm-web-01
```bash
ssh azureuser@<public-ip-of-vm-web-01>
```
| Result | Status |
|--------|--------|
| SSH connection established | ✅ PASSED |

---

### Test 2 — Ping vm-db-01 from vm-web-01
```bash
ping 10.0.2.4
```
| Result | Status |
|--------|--------|
| ICMP replies received from 10.0.2.4 | ✅ PASSED |
| VNet internal routing confirmed working | ✅ PASSED |

---

### Test 3 — Verify vm-db-01 NOT reachable from internet
```bash
ssh azureuser@10.0.2.4  # attempted from local machine
```
| Result | Status |
|--------|--------|
| Connection timed out — no public access | ✅ PASSED |
| NSG blocking confirmed working | ✅ PASSED |

---

## 5. Cost Management Configuration

| Setting | Value |
|---------|-------|
| Budget Alert | $10 monthly threshold |
| Alert Recipients | Account email |
| Auto-Shutdown — vm-web-01 | Enabled (daily) |
| Auto-Shutdown — vm-db-01 | Enabled (daily) |
| Resource Tagging | project=vnet-project-001 |

**Why cost management matters:**
Even in a learning environment, idle VMs running 24/7 accumulate costs. Auto-shutdown ensures VMs are not running when not in use. Budget alerts notify you before spending exceeds your threshold.

---

## 6. NSG Traffic Flow Summary

```
[INTERNET]
    |
    |── SSH (Port 22)  ──────────► [nsg-web-subnet] ──► [vm-web-01]
    |── HTTP (Port 80) ──────────► [nsg-web-subnet] ──► [vm-web-01]
    |
    |── Any traffic ─────────────► [nsg-db-subnet]  ──► ✗ BLOCKED
    |
[vm-web-01] (10.0.1.4)
    |
    |── Internal traffic ────────► [nsg-db-subnet]  ──► [vm-db-01] ✅
         (from 10.0.1.0/24)              |                (10.0.2.4)
                                   Allow-From-WebSubnet
```

---

## 7. Key Learnings

**VNet Segmentation**
Separating resources into subnets provides network-level isolation. Even within the same VNet, subnets can have completely different security policies applied via NSGs.

**NSG Priority System**
Lower priority numbers are evaluated first. Always place specific allow rules at lower numbers (100, 110) before the default deny-all rule (65500).

**Database Security**
A database server should never have a public IP. All access should be routed through the application tier. This is a fundamental cloud security principle.

**Cost Awareness**
Auto-shutdown and budget alerts are not optional — they are essential habits for any cloud developer working with free or limited subscriptions.

**Region Consistency**
All resources in a project must be deployed in the same region. Mismatched regions cause connectivity failures and unexpected latency.

---

## 8. Tools Used

| Tool | Purpose |
|------|---------|
| Azure Portal | Resource creation and management |
| MacBook Terminal (zsh) | SSH testing and ping validation |
| Azure Cost Management | Budget alerts and cost tracking |
| GitHub | Project documentation and portfolio |
| draw.io / Claude | Network architecture diagram |

---

## 9. Resources & References

- [Azure Virtual Network Documentation](https://docs.microsoft.com/en-us/azure/virtual-network/)
- [Network Security Groups Overview](https://docs.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [Azure VM Pricing](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/)
- [Azure Cost Management](https://docs.microsoft.com/en-us/azure/cost-management-billing/)

---

*Documentation created as part of structured Azure learning journey — Phase 1, Week 2*
*Next Project: Serverless App Deployment using Azure Functions*
