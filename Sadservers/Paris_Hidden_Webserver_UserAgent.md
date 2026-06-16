# Scenario: "Paris": Where is my webserver?

**Level:** Medium  
**Tags:** `http-headers`, `user-agent`, `python`, `reconnaissance`, `troubleshooting`  

## 📝 Problem Description
A development security credential (password) is served by a hidden local web server listening on `localhost:5000`. Standard identification scans via unconfigured client utilities drop connection attempts or withhold the target string. The goal is to safely extract the hidden value and commit it to `/home/admin/mysolution` under tight non-root privilege constraints.

## 💡 The Solution Strategy
Because standard system diagnostics like `ip netns` and network routing interfaces appear uniform, the block lives at the application layer. Web servers frequently inspect incoming HTTP headers to prevent programmatic scraping or unauthorized api crawling. 

By running an unmasked process audit (`ps aux`), the web server is identified as a custom Python script (`webserver.py`). The "trick" is bypassing its header-filtering logic by passing a custom spoofed **User-Agent string** (`-A "Mozilla/5.0"`) via `curl`, masking our terminal script request as an authentic human web browser connection.

### The Commands
```bash
# 1. Probe the target port directly to analyze baseline transport layer handling
curl -iv http://localhost:5000

# 2. Audit active user-space processes to isolate the handling daemon binary
ps aux | grep -iE 'server|app|python|5000'
# Output reveals: PID 763 -> /usr/bin/python3 /home/admin/webserver.py running as root

# 3. Spoof an authentic browser signature to bypass application-layer scraping filters
curl -A "Mozilla/5.0" http://localhost:5000
# Output cleanly returns the plain-text password string

# 4. Save the password directly to the validation target file path
echo "YOUR_EXTRACTED_PASSWORD" > /home/admin/mysolution

# 5. Validate string formatting against the target cryptographic signature
md5sum /home/admin/mysolution
# Output returns: d8bee9d7f830d5fb59b89e1e120cce8e (Success)
