# Scenario: "Geneva": Renew an SSL Certificate

**Level:** Easy  
**Tags:** `ssl`, `tls`, `nginx`, `openssl`, `security`, `troubleshooting`  

## 📝 Problem Description
An Nginx web server is configured to serve a simple web page over HTTPS, but client connections are failing because the installed SSL/TLS certificate has expired. 

The objective is to restore secure traffic by generating and installing a brand-new valid certificate. To pass operational constraints, the newly forged certificate must retain the exact same cryptographic "Subject" identity (Country, State, Locality, Organization, Organizational Unit, and Common Name) as the expired one.

## 💡 The Solution Strategy
Resolving expired self-signed or internal certificates requires a clear discovery and replacement pipeline. 
1. **Reconnaissance:** We use `openssl s_client` to interrogate the live web server and extract the exact Subject signature of the dying certificate. We then grep the Nginx configuration directory to pinpoint exactly where the `.crt` and `.key` files are stored on the disk.
2. **Forging:** We utilize `openssl req` to generate a fresh RSA keypair and a new X.509 certificate valid for 365 days, explicitly passing the extracted Subject string so the identity remains unbroken. By targeting the output flags directly to the paths found in step 1, we overwrite the bad files in place.
3. **Daemon Reload:** Finally, we signal the Nginx master process to reload its configuration, purging the expired certificate from RAM and loading the newly generated assets from disk.

### The Commands
```bash
# 1. Interrogate the local web server to extract the exact Subject string of the expired certificate
echo | openssl s_client -connect localhost:443 2>/dev/null | openssl x509 -noout -subject
# Output yields: subject=CN = localhost, O = Acme, OU = IT Department, L = Geneva, ST = Geneva, C = CH

# 2. Search the Nginx configuration tree to locate the physical file paths for the SSL assets
sudo grep -R "ssl_certificate" /etc/nginx/
# Output reveals the active paths: /etc/nginx/ssl/nginx.crt and /etc/nginx/ssl/nginx.key

# 3. Generate a new 365-day self-signed certificate overwriting the old files in place
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/nginx.key \
  -out /etc/nginx/ssl/nginx.crt \
  -subj "/C=CH/ST=Geneva/L=Geneva/O=Acme/OU=IT Department/CN=localhost"

# 4. Gracefully reload the Nginx daemon to serve the newly minted files
sudo systemctl reload nginx

# 5. Verify the TLS handshake and HTTP response locally
curl -kv https://localhost
# Output confirms a valid 200 OK response and an updated 2027 expiration date
