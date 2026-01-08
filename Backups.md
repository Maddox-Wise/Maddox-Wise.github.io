# Automated Backups with Borgmatic

**Timeframe:** 2025  
**Tools:** BorgBackup, Borgmatic, Cron, Discord Webhooks  
**Environment:** Linux homelab server (self-hosted infrastructure)

---

## Objective

Design and implement a **secure, automated backup solution** to protect critical system and application data from loss due to hardware failure, misconfiguration, or user error. The goal was to ensure backups run without manual intervention while providing visibility into backup health and recoverability.

---

## Implementation

- Deployed **BorgBackup** as the underlying backup engine for:
  - Block-level deduplication to reduce storage usage
  - Strong encryption to protect data at rest
  - Compression to improve storage efficiency
- Used **Borgmatic** as an automation and configuration layer to:
  - Define backup sources and repositories declaratively
  - Manage encryption settings and passphrases securely
  - Apply retention policies for daily, weekly, and monthly archives
- Configured **Cron** jobs to:
  - Run backups automatically during low-usage hours
  - Ensure consistent execution without user intervention

---

## Monitoring & Alerts

- Integrated **Discord webhooks** for real-time monitoring
- Automated notifications include:
  - Backup success or failure status
  - Total execution duration
  - Size of the latest archive
  - Total repository size after deduplication
- Enabled rapid response to failed or stalled backup jobs

---

## Validation

- Performed **regular archive inspections** using Borg commands to:
  - Verify archive integrity
  - Confirm encryption and deduplication statistics
- Tested **restore workflows** by:
  - Restoring individual files and directories
  - Validating restored data integrity and permissions
- Confirmed backups are usable in disaster recovery scenarios

---

## Outcomes

- Gained hands-on experience designing and maintaining automated backup systems
- Developed a practical understanding of:
  - Backup retention strategies
  - Storage optimization through deduplication
  - Disaster recovery planning
- Increased confidence in restoring data quickly and reliably during system failures
