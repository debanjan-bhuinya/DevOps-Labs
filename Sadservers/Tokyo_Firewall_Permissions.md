# Scenario: "Tokyo": Web server unreached

**Level:** Medium  
**Tags:** `linux`, `networking`, `iptables`, `firewall`, `permissions`  

## 📝 Problem Description
A local web server is running, but requests to `curl 127.0.0.1:80` hang indefinitely. The issue spans multiple layers of the system, involving both network-level restrictions (firewall) and application-level constraints (file permissions).

## 💡 The Solution Strategy
Troubleshooting a silent network failure requires verifying transit rules first (the firewall), followed by checking service access configurations (permissions) once the data packets can actually reach the application layer.

### The Commands
```bash
# 1. Inspect the netfilter firewall rules
sudo iptables -L
# Output reveals an active rule in the INPUT chain dropping traffic to tcp dpt:http.

# 2. Flush all active iptables chains to clear the network block
sudo iptables -F

# 3. Test the HTTP connection again
curl 127.0.0.1:80
# The connection succeeds, but returns a "403 Forbidden" error page.

# 4. Audit the file permissions of the target document
sudo ls -l /var/www/html/index.html
# Output shows strict root-only permissions (-rw-------), preventing the web daemon from reading it.

# 5. Grant universal read permission to the web file
sudo chmod 644 /var/www/html/index.html

# 6. Verify the final successful resolution
curl 127.0.0.1:80
# Output successfully returns: hello sadserver
