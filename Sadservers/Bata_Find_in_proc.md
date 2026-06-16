# Scenario: "Bata": Find in /proc

**Level:** Easy  
**Tags:** `bash`, `proc`, `sysctl`, `text-manipulation`, `troubleshooting`  

## 📝 Problem Description
A hidden asset (password) has been injected into a runtime configuration file under the virtual filesystem path `/proc/sys`. The target file contents are known to initiate with the strict prefix string `secret:`. 

The objective is to recursively audit the target tree, isolate the matching parameter file, extract the explicit value following the delimiter, and commit it to `/home/admin/secret.txt` under restrictive non-root privilege constraints.

## 💡 The Solution Strategy
The `/proc` and `/proc/sys` directories are pseudo-filesystems that act as an interface to internal kernel data structures rather than real files on physical media blocks. Because many of these operational hooks require root elevation, scanning the tree as a standard user triggers massive standard error telemetry bursts (`Permission denied`).

The optimal engineering fix is executing a recursive string search (`grep -r`) across the sub-tree path while explicitly binding the standard error file descriptor (`2`) over to the Linux null data sink (`/dev/null`). This isolates successful data extractions instantly.

### The Commands
```bash
# 1. Recursively search the kernel parameter tree while silencing permission errors
grep -r "secret:" /proc/sys/ 2>/dev/null
# Output yields file path and string layout matching the target pattern

# 2. Extract the string following the colon delimiter and write it to the solution target
echo "YOUR_EXTRACTED_PASSWORD" > /home/admin/secret.txt

# 3. Validate formatting accuracy using the target cryptographic MD5 validation check
md5sum /home/admin/secret.txt
# Output returns: a7fcfd21da428dd7d4c5bb4c2e2207c4 (Success)
