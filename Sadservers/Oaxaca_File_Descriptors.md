# Scenario: "Oaxaca": Close an Open File

**Level:** Medium  
**Tags:** `linux`, `bash`, `file-descriptors`, `troubleshooting`  

## 📝 Problem Description
A file located at `/home/admin/somefile` is open for writing by an active process. The objective is to force the system to close this file handle so that `lsof` returns nothing, but with a strict constraint: you are **not allowed to kill the process** holding it open.

## 💡 The Solution Strategy
When a file is held open, standard protocol is to terminate or restart the owning daemon. If that process is un-killable (e.g., a critical system shell), you must manipulate the process's file descriptors directly. If the process happens to be your own active Bash session, you can utilize native shell redirection parameters to close the stream.

### The Commands
```bash
# 1. Audit the file to find the owning process and file descriptor
lsof /home/admin/somefile
# Output reveals the command is 'bash' (the active shell) and the FD is '77w'.

# 2. Force the active shell to drop the file descriptor without terminating the session
exec 77>&-

# 3. Verify the file descriptor has been freed
lsof /home/admin/somefile
# Output returns nothing (Success).

