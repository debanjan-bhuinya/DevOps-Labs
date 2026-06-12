# Scenario: "Lisbon": etcd SSL cert troubles

**Level:** Medium  
**Tags:** `etcd`, `ssl`, `iptables`, `networking`, `troubleshooting`  

## 📝 Problem Description
The objective is to retrieve the value of the key `foo` from a secure `etcd` server cluster. Out of the box, `etcdctl get foo` fails due to an x509 transport layer security (TLS) certificate validation error. Once the certificate block is cleared, a hidden network engineering trap prevents direct communication with the database engine.

## 💡 The Solution Strategy
This scenario presents a layered defensive trap: an expired security certificate combined with an aggressive kernel-level port redirection rule in the firewall configuration. The resolution requires manipulating the OS clock layer to satisfy TLS validity windows, followed by a complete flush of the netfilter NAT tables.

### The Commands
```bash
# 1. Inspect the initial cluster error state
etcdctl get foo
# Error signature: x509: certificate has expired or is not yet valid

# 2. Inspect the daemon arguments to determine file locations and constraints
sudo systemctl status etcd
# Reveals the cluster is bound to https://localhost:2379 using local certs

# 3. Roll back the system clock to a period where the cert was active (Cert expired Jan 2023)
sudo date -s "2023-01-20"

# 4. Re-run the command to hit the hidden secondary network trap
etcdctl get foo
# Error signature: client: response is invalid json (due to rogue Nginx redirect interception)

# 5. Clear the malicious NAT table rules diverting port 2379 traffic to port 443
sudo iptables -t nat -F

# 6. Verify successful retrieval of the target key
etcdctl get foo
# Output cleanly returns:
# foo
# bar
