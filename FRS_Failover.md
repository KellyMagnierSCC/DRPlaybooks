# 🛠️ SQL Server Always On Availability Group – DR Failover Runbook

## 📍 Overview

This document describes the **manual failover** to the DR site for a SQL Server Always On Availability Group deployed across two data centers using Windows Server Failover Clustering (WSFC).

---
## ⚙️ Configuration
**Cluster**: `ODC1-CLS-13`  
**Availability Group**: SFRS_AG
**Availability Group Listener**: FRS_AGL
**Primary Nodes**: `ODC1-SQL-13`, `ODC1-SQL-14`  
**DR Node**: `ODC2-SQL-13` (Async Replica, `NodeWeight = 0`)  
**Quorum Witness**: File Share Witness  
**AG Mode**: Synchronous on primary nodes, Asynchronous on DR node

---

## 🔥 Disaster Recovery (DR) Failover Runbook

---

### 🧭 Purpose

This runbook outlines the step-by-step procedure to manually fail over the Availability Group to the DR node in the event that both primary site nodes (ODC1-SQL-13 and ODC1-SQL-14) become unavailable.

---

###  ⚠️ Preconditions

- Primary site is **fully down** or unreachable.
- DR node `ODC2-SQL-13` is online and healthy.
- All stakeholders agree that DR mode should be activated.
- Acceptable risk of **data loss** if DR replica is asynchronous.
- This process assumes manual intervention is required due to quorum loss.

---

### ✅ Summary of Steps
1. Start DR cluster node with ForceQuorum
2. Validate cluster and AG services are running
3. Manually force failover of the Availability Group to DR
4. Validate database status and availability
5. Notify stakeholders and proceed with DR operations

---
### 🪛 Step-by-Step Instructions

1. **Connect to the DR Node**

    Log on to `ODC2-SQL-13`.

   *Use RDP or a secure console to access the DR node.*


2. **Force Cluster Quorum on DR Node**

    ```powershell
    Start-ClusterNode -Name ODC2-SQL-13 -FixQuorum
    ```
    
> [!CAUTION]
> This starts the cluster without quorum. Only do this if you're sure the primary site is fully down and won't cause a split-brain.


 3. **Confirm Cluster is Running**
 ```powershell
Get-ClusterNode
 ```
  *Ensure ODC2-SQL-13 shows Up.*


4. **Start Cluster Service (if stopped)**

    ```powershell
    Start-Service -Name ClusSvc
    ```


5. **Bring AG Online and Force Failover**

    ```powershell
    Import-Module SqlServer
    Set-SqlAvailabilityGroup -Path "SQLSERVER:\SQL\ODC2-SQL-13\DEFAULT\AvailabilityGroups\SFRS_AG" -Failover
    ```


6. **Validate Availability Group and Database Health**

     ```powershell
    Get-SqlAvailabilityGroup -ServerInstance ODC2-SQL-13
    Get-SqlAvailabilityReplica -GroupName SFRS_AGL -ServerInstance ODC2-SQL-13

    ```
  Check in SSMS or with T-SQL:
    ```sql
    SELECT ag.name, drs.database_name, drs.synchronization_state_desc
    FROM sys.dm_hadr_database_replica_states drs
    JOIN sys.availability_groups ag ON drs.group_id = ag.group_id;
    ```

7. **Bring Listener Online (if needed)**

    ```powershell
    Get-ClusterResource
    Start-ClusterResource "FRS_AGL"
    ```

8. **Notify Stakeholders**

    - DR is now active.
    - Applications should point to listener or DR-specific DNS.

---
## 📋 Validation Checklist

| Validation Step                                   | Status |
|---------------------------------------------------|--------|
| Cluster started on ODC2-SQL-13                    | ☐      |
| ODC2-SQL-13 has Quorum                            | ☐      |
| Availability Group is Primary on DR               | ☐      |
| Databases are `SYNCHRONIZED`                      | ☐      |
| Listener is Online and responding to connections  | ☐      |
| Apps and Users Reconnected Successfully           | ☐      |
| Backups and Jobs are Running Normally             | ☐      |

---

