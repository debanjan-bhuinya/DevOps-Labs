# Scenario: "Salta": Docker container won't start

**Level:** Medium  
**Tags:** `linux`, `docker`, `troubleshooting`, `networking`, `permissions`  

## 📝 Problem Description
A Node.js web application needs to be containerized and exposed on port `8888`. The environment has restricted permissions, a broken `Dockerfile`, and a hidden decoy service actively blocking the required network port.

## 💡 The Solution Strategy
Troubleshooting this scenario requires a multi-step debugging pipeline: bypassing Docker socket permission errors, fixing application-level typos, and hunting down OS-level network conflicts.

### The Commands
```bash
# 1. Inspect the Dockerfile and fix the fatal typo
nano Dockerfile
# Changed the final CMD from ["node", "serve.js"] to ["node", "server.js"] to match the actual file.

# 2. Identify the process hogging the required port (8888)
sudo ss -tulpn | grep 8888
# Output reveals a hidden Nginx service actively listening on port 8888.

# 3. Terminate the rogue decoy processes
sudo kill -9 <PID_1> <PID_2> <PID_3>
# (Replaced <PIDs> with the specific Nginx Process IDs found in step 2)

# 4. Build the Docker image (using sudo to bypass the /var/run/docker.sock permission denied error)
sudo docker build -t salta-app .

# 5. Purge any broken or dead containers to satisfy the "only one container" rule
sudo docker rm -f $(sudo docker ps -aq)

# 6. Run the new container in the background, mapping the host port to the container port
sudo docker run -d -p 8888:8888 --name myapp salta-app

# 7. Verify the final successful resolution
curl localhost:8888
# Output successfully returns: Hello World!
