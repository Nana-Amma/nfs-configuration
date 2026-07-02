Change Proposal: Add port=2049 to Automounter Configuration

1. Summary
Objective: Add `port=2049` to `/etc/auto.resstore` on all login nodes to improve NFS mounting stability.

2. Implementation Details
Target Nodes
| Node Type | Affected | Change Required |
| :--- | :--- | :--- |
| login1 | Yes | Modify /etc/auto.resstore |
| login2 | Yes | Modify /etc/auto.resstore |
| login3 | Yes | Modify /etc/auto.resstore |

Configuration Change
Before:
b0069   -soft,timeo=10,retrans=3,acl,noresvport   svmt003.leeds.ac.uk:/b0069

After:
b0069   -soft,timeo=10,retrans=3,acl,noresvport,port=2049   svmt003.leeds.ac.uk:/b0069

3. Implementation Steps
1. Backup: sudo cp /etc/auto.resstore /etc/auto.resstore.backup.$(date +%Y%m%d)
2. Apply Change: sudo sed -i s/noresvport/noresvport,port=2049/g /etc/auto.resstore
3. Verify Diff: diff /etc/auto.resstore.backup.$(date +%Y%m%d) /etc/auto.resstore
4. Reload Service: sudo systemctl reload autofs
5. Initial Verification: time quota -s

4. Testing & Validation Plan
- Pre-Implementation: Verify issue on a single login node using time quota -s.
- Validation Commands:
  - Verify mount options: mount -t nfs,nfs4 | grep /resstore
  - Check for port 111 connections (should be zero): sudo ss -an | grep ":111" | grep ESTABLISHED
- Rollback Plan: 
  - Restore from backup: sudo cp /etc/auto.resstore.backup.[DATE] /etc/auto.resstore
  - Reload: sudo systemctl reload autofs

5. Risk Assessment
| Risk Level | Description | Mitigation |
| :--- | :--- | :--- |
| Low | NFS mount failure | Backup in place; immediate rollback available. |
| Low | Service disruption | Reloading autofs is non-disruptive; can be scheduled for maintenance. |
