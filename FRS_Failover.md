# 🛠️ SQL Server Always On Availability Group – DR Failover & Failback Runbook

**Cluster**: `ODC1-CLS-13`  
**Primary Nodes**: `ODC1-SQL-13`, `ODC1-SQL-14`  
**DR Node**: `ODC2-SQL-13` (Async Replica, `NodeWeight = 0`)  
**Quorum Witness**: File Share Witness  
**AG Mode**: Synchronous on primary nodes, Asynchronous on DR node

---

## 📍 Overview

This document describes the **manual failover** to the DR site and **failback** to the primary site for a SQL Server Always On Availability Group deployed across two data centers using Windows Server Failover Clustering (WSFC).

---

## 🔥 Disaster Recovery (DR) Failover Procedure

### ✅ Preconditions

- Primary site is **fully down** or unreachable.
- DR node `ODC2-SQL-13` is online and healthy.
- All stakeholders agree that DR mode should be activated.
- Acceptable risk of **data loss** if DR replica is asynchronous.

### 🪛 Steps

1. **Connect to the DR Node**

    Log on to `ODC2-SQL-13`.

2. **Force Cluster Quorum on DR Node**

    ```powershell
    Start-ClusterNode -Name ODC2-SQL-13 -FixQuorum
    ```

3. **Start Cluster Service (if stopped)**

    ```powershell
    Start-Service -Name ClusSvc
    ```

4. **Bring AG Online and Force Failover**

    ```powershell
    Import-Module SqlServer
    Set-SqlAvailabilityGroup -Path "SQLSERVER:\SQL\ODC2-SQL-13\DEFAULT\AvailabilityGroups\SFRS_AG" -Failover
    ```

5. **Validate Availability Group and Database Health**

    ```sql
    SELECT ag.name, drs.database_name, drs.synchronization_state_desc
    FROM sys.dm_hadr_database_replica_states drs
    JOIN sys.availability_groups ag ON drs.group_id = ag.group_id;
    ```

6. **Bring Listener Online (if needed)**

    ```powershell
    Start-ClusterResource "FRS_AGL"
    ```

7. **Notify Stakeholders**

    - DR is now active.
    - Applications should point to listener or DR-specific DNS.

---

