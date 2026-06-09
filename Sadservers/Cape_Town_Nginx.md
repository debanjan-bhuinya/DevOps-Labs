# Scenario: "Cape Town": Borked Nginx

**Level:** Medium  
**Tags:** `linux`, `nginx`, `systemd`, `troubleshooting`  

## 📝 Problem Description
An Nginx web server is failing to serve the default page. Initially, it returns a "Connection refused" error indicating the service is dead. After fixing the initial boot failure, it returns an "HTTP 500 Internal Server Error", indicating a runtime crash when attempting to process a request. 

## 💡 The Solution Strategy
Web server troubleshooting follows a strict pipeline: first, validate the configuration syntax. Second, check the application error logs. Third, audit the service manager (systemd) limits and environment variables.

### The Commands
```bash
# 1. Test Nginx configuration syntax to find the boot failure
sudo nginx -t
# Output points to an unexpected ";" on line 1 of the default site config.

# 2. Fix the syntax error
sudo nano /etc/nginx/sites-enabled/default
# (Remove the stray semicolon, save, and exit)

# 3. Restart the service and test
sudo systemctl restart nginx
curl -Is 127.0.0.1:80 | head -1
# Output shows a new error: HTTP/1.1 500 Internal Server Error

# 4. Check the application logs to diagnose the 500 error
sudo tail -n 10 /var/log/nginx/error.log
# Output reveals: "failed (24: Too many open files)"

# 5. Locate and fix the systemd override file choking the service
sudo nano /etc/systemd/system/nginx.service
# Change LimitNOFILE=10 to LimitNOFILE=65535

# 6. Reload the system daemon and restart Nginx
sudo systemctl daemon-reload
sudo systemctl restart nginx

# 7. Final verification
curl -Is 127.0.0.1:80 | head -1
# Output returns: HTTP/1.1 200 OK
