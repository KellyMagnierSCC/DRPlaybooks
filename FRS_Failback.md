## 🔁 Primary Site Failback Procedure

### ✅ Preconditions

- Primary nodes (`ODC1-SQL-13`, `ODC1-SQL-14`) are online and healthy.
- Network connectivity is restored.
- AG synchronization is complete between DR and primary.
- All teams have approved failback.

### 🪛 Steps

1. **Re-enable Node Weights for Quorum (optional)**

    ```powershell
    (Get-ClusterNode -Name "ODC1-SQL-13").NodeWeight = 1
    (Get-ClusterNode -Name "ODC1-SQL-14").NodeWeight = 1
    (Get-ClusterNode -Name "ODC2-SQL-13").NodeWeight = 0
    ```

2. **Start Cluster Services (if required)**

    ```powershell
    Start-ClusterNode -Name ODC1-SQL-13
    Start-ClusterNode -Name ODC1-SQL-14
    ```

3. **Validate AG Synchronization**

    ```sql
    SELECT ag.name, drs.replica_server_name, drs.database_name,
           drs.synchronization_state_desc, drs.synchronization_health_desc
    FROM sys.dm_hadr_database_replica_states drs
    JOIN sys.availability_groups ag ON drs.group_id = ag.group_id;
    ```

4. **Manual Failover Back to Primary Node**

    ```powershell
    Set-SqlAvailabilityGroup -Path "SQLSERVER:\SQL\ODC1-SQL-13\DEFAULT\AvailabilityGroups\SFRS_AG" -Failover
    ```

5. **Validate Listener and Application Connectivity**

    ```powershell
    Get-ClusterResource "FRS_AGL"
    Start-ClusterResource "FRS_AGL"  # if required
    ```

6. **Update Monitoring and Backup Jobs**

7. **Notify All Stakeholders of Recovery**

---

## 📋 Validation Checklist

| Validation Step                                   | Status |
|---------------------------------------------------|--------|
| DR or Primary Cluster Node has Quorum             | ☐      |
| Availability Group is Primary on correct node     | ☐      |
| Databases are `SYNCHRONIZED`                      | ☐      |
| Listener is Online                                | ☐      |
| Apps and Users Reconnected Successfully           | ☐      |
| Backups and Jobs are Running Normally             | ☐      |

---

## 👥 RACI Matrix

| Task                                         | DBA Team | SysAdmin Team | Infra/Network Team |
|----------------------------------------------|----------|----------------|---------------------|
| Assess and declare DR event                  | A        | C              | C                   |
| Force quorum on DR node                      | R        | A              | C                   |
| Manual AG failover to DR                     | A        | R              | C                   |
| Re-enable cluster nodes at primary           | R        | A              | C                   |
| Confirm replication and synchronization      | A        | C              | C                   |
| Manual AG failback to primary                | A        | R              | C                   |
| Update node weights and quorum configuration | R        | A              | C                   |
| Test application connectivity post-failback  | A        | C              | C                   |
| Notify stakeholders and document changes     | A        | R              | R                   |

**R** = Responsible  
**A** = Accountable  
**C** = Consulted

---

## 📘 References

- `FailoverClusters` PowerShell Module  
- `SqlServer` PowerShell Module  
- WSFC Logs:  
  ```powershell
  Get-ClusterLog -UseLocalTime -Destination C:\ClusterLogs
