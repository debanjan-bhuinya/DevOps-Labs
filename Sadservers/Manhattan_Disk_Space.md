# Scenario: "Manhattan": can't write data into database

**Level:** Medium  
**Tags:** `linux`, `troubleshooting`, `postgres`, `disk-space`  

## 📝 Problem Description
An existing PostgreSQL database is failing to accept `INSERT` queries. The issue is explicitly stated to be OS-level rather than a specific Postgres configuration error. We have full `root` access to diagnose and resolve the issue.

## 💡 The Solution Strategy
When a database suddenly stops writing data and it isn't a configuration issue, the root cause is almost always OS resource exhaustion—specifically, a filesystem reaching 100% capacity. The strategy is to check disk usage, identify the mount point holding the database, find the files hogging the space, and clear them out so the database service can recover.

### The Commands
```bash
# 1. Check human-readable disk space across all mounted filesystems
df -h
# Output reveals that /dev/nvme0n1 (mounted on /opt/pgdata) is at 100% Use.

# 2. Scan the specific full directory for the largest files
sudo du -ah /opt/pgdata | sort -rh | head -10
# Output reveals two massive, unnecessary dummy backup files: file1.bk and file2.bk.

# 3. Remove the space-hogging files
sudo rm /opt/pgdata/file1.bk /opt/pgdata/file2.bk

# 4. Restart the PostgreSQL service so it can recover from its locked state
sudo systemctl restart postgresql

# 5. Run the test query to verify the database can write rows again
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
