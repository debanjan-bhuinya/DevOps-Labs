# Scenario: "Venice": Am I in a container?

**Level:** Medium  
**Tags:** `linux`, `containers`, `security`, `troubleshooting`  

## 📝 Problem Description
The objective is to investigate the host environment and determine whether the active shell is running inside a full Virtual Machine (VM) or an isolated Container environment. There is no automated testing script for this scenario; it relies entirely on manual system administration forensics.

## 💡 The Solution Strategy
To differentiate a container from a VM, you must look for virtualization fingerprints left behind by the host kernel. This involves checking specialized systemd utilities, process control groups (cgroups), and hidden runtime files.

### The Commands
```bash
# 1. Query the systemd virtualization detector (The most reliable method on modern systemd OSs)
systemd-detect-virt
# Output returned: "container-other" 
# (Conclusion: We are in a container)

# 2. Inspect the control groups assigned to PID 1 (The init process)
cat /proc/1/cgroup
# Output returned: "0::/init.scope"
# (Conclusion: PID 1 is running systemd. While normal Docker containers show "docker" in this output, System Containers like LXC or systemd-nspawn boot full init systems, masking standard application-container fingerprints).
