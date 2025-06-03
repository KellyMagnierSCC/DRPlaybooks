# ✅ Split-Brain Prevention Checklist – DR Failover Safety

> [!IMPORTANT]
> Use this checklist before initiating a manual failover to your **DR (secondary site)** in an Always On Availability Group (AG) with asynchronous replica(s) and a multi-site Windows Failover Cluster.

---

## 🔹 1. Confirm the Primary Site is Truly Unavailable

| Check | Description | Confirmed |
|-------|-------------|-----------|
| 🔌 Network is down | No connectivity to both primary replicas from DR site | ☐ |
| 📡 Primary nodes are unreachable | You cannot ping or RDP to primary replicas (`ODC1-SQL-XX`, `ODC1-SQL-XX`) | ☐ |
| 🧭 No response from listener | AG listener does not respond from DR site or application side | ☐ |
| 🛑 Application outage verified | Confirm outage with application and business owners | ☐ |
| 🚫 AG is not reachable via SQL | `sqlcmd`, SSMS, or PowerShell cannot connect to AG primary | ☐ |

---

## 🔹 2. Confirm Quorum Is Lost or Unavailable at Primary Site

| Check | Description | Confirmed |
|-------|-------------|-----------|
| ❌ Cluster does not have quorum | Primary site cannot maintain cluster quorum (check via monitoring or logs) | ☐ |
| 🔇 Witness is not accessible from primary site | File Share Witness is designed to be reachable only from the DR site (optional best practice) | ☐ |

---

## 🔹 3. Verify DR Site Can Take Over Without Risk

| Check | Description | Confirmed |
|-------|-------------|-----------|
| 🧱 DR node is isolated (NodeWeight = 0)** | Prevents DR node from forming quorum on its own unless explicitly forced (`-FixQuorum`) | ☐ |
| 📉 DR replica is healthy | Confirm the async secondary (`ODC2-SQL-XX`) is in a RESTORING state and was synchronizing recently | ☐ |
| 🗂 Databases are in a restorable state | Review `sys.dm_hadr_database_replica_states` to confirm log chain is intact | ☐ |

---

## 🔹 4. Prepare and Execute Controlled Failover

| Check | Description | Confirmed |
|-------|-------------|-----------|
| 🔐 Authorized DR procedure invoked | DR failover is approved per policy / RACI matrix | ☐ |
| 🔧 `Start-ClusterNode -FixQuorum` used intentionally | Force quorum only on DR node and ensure no other nodes can contend for ownership | ☐ |
| 🧭 AG failover is manual, using PowerShell or SSMS | Example: `Set-SqlAvailabilityGroup -Failover` or SSMS failover wizard | ☐ |
| 🔌 Listener brought online manually if needed | `Start-ClusterResource "<ListenerName>"` | ☐ |
| 📢 Teams notified | All relevant stakeholders and teams (DBA, sysadmin, app owners) are informed | ☐ |

---

## 🧯 Post-Event Cleanup (When Primary Site Recovers)

| Check | Description | Confirmed |
|-------|-------------|-----------|
| 🔄 Stop cluster on recovered primary nodes (optional) | Prevent accidental rejoin until fully validated | ☐ |
| 🔍 Validate data consistency | Compare transaction history between primary and DR (if possible) | ☐ |
| 🔁 Resync replicas manually | Reconfigure AG, remove and re-add replicas if needed | ☐ |
| 🔄 Fail back to original site (optional) | Only when safe and stable | ☐ |

---

## 📎 Tips

- Document every step taken during the failover.
- Use the DR **runbook** in conjunction with this checklist.
- Keep this checklist in your Git repo.
- Assign responsibility using the **RACI matrix** included in your DR plan.
