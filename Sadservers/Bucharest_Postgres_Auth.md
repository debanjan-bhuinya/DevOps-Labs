# Scenario: "Bucharest": Connecting to Postgres

**Level:** Easy  
**Tags:** `postgres`, `database`, `networking`, `troubleshooting`  

## 📝 Problem Description
A web application is failing to connect to a PostgreSQL 13 database cluster running on localhost (`127.0.0.1`), throwing a fatal error indicating that the connection was explicitly rejected by the database's internal security layer despite using valid credentials.

## 💡 The Solution Strategy
PostgreSQL access is controlled at the network layer via the Host-Based Authentication configuration file (`pg_hba.conf`). Troubleshooting requires checking the service availability, verifying network sockets, and evaluating the order of evaluation inside the security configuration.

### The Commands
```bash
# 1. Run the connection test to confirm the exact error message
PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q'
# Error: FATAL: pg_hba.conf rejects connection for host "127.0.0.1"

# 2. Confirm the database daemon is active and listening on the network
sudo systemctl status postgresql
sudo ss -tulpn | grep 5432
# Output shows PostgreSQL running and bound cleanly to 127.0.0.1:5432

# 3. Modify the host-based authentication firewall layout
sudo nano /etc/postgresql/13/main/pg_hba.conf

# 4. Comment out the explicit blanket reject traps at the top of the file:
# host    all             all             all                     reject
# host    all             all             all                     reject

# 5. Ensure the fallback local rule is enabled for password validation
host    all             all             127.0.0.1/32            md5

# 6. Cycle the database service to reload the updated parameters
sudo systemctl restart postgresql

# 7. Rerun the test to confirm access is granted
PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q'
# Output exits silently with code 0 (Success)
