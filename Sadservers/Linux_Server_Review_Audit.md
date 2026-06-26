# Scenario: Linux Server Review - Guided Learning

**Level:** Easy  
**Tags:** `linux`, `auditing`, `monitoring`, `troubleshooting`, `reconnaissance`  

## 📝 Problem Description
This is an exploratory, open-ended scenario designed for guided learning rather than a specific break-fix validation. 

The objective is to practice the critical first five minutes of incident response or system administration: logging into a completely undocumented server and accurately determining its primary purpose, hardware health (CPU, RAM, Disk), and active processing footprint without relying on external monitoring tools.

## 💡 The Solution Strategy
When dropped into a blind environment, a systematic profiling strategy is required to build situational awareness. The methodology is broken down into three distinct phases using native POSIX utilities:
1. **Resource Profiling:** Assess the core hardware metrics to ensure the machine isn't actively crashing or resource-starved.
2. **Network & Identity Profiling:** Map open listening ports to deduce the server's primary role (e.g., a web server, a database, a cache layer).
3. **Process Profiling:** Sort the active process tree by resource consumption to identify exactly which binaries are driving the workload.

### The Standard Audit Commands
```bash
# === Phase 1: Hardware & Resource Utilization ===

# 1. Check system uptime and 1, 5, and 15-minute CPU load averages
uptime

# 2. Audit memory allocation, available RAM, and active swap space usage
free -h

# 3. Verify block storage capacity and partition mount thresholds
df -h

# === Phase 2: Identity & Network Footprint ===

# 4. Enumerate all active listening TCP/UDP ports and their bound service daemons
sudo ss -tulpn

# === Phase 3: Process Profiling ===

# 5. Isolate the top 10 most memory-intensive processes currently running
ps aux --sort=-%mem | head -n 10

# 6. Isolate the top 10 most CPU-intensive processes currently running
ps aux --sort=-%cpu | head -n 10
