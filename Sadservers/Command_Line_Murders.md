# Scenario: "The Command Line Murders" (SadServers Twist)

**Level:** Easy  
**Tags:** `bash`, `scripting`, `awk`, `md5sum`  

## 📝 Problem Description
The objective is to find the name of a murderer from a list of suspects. Unlike the traditional "Command Line Murders" investigation which requires cross-referencing multiple files with `grep`, this specific scenario provides a shortcut: the exact MD5 hash of the murderer's name.

**Target Hash:** `9bba101c7369f49ca890ea96aa242dd5`
**Requirement:** Write the correct name into `~/mysolution`.

## 💡 The Solution strategy
Instead of manually investigating the crime scene files, we can write a Bash script to brute-force the target hash against the list of names provided in the `people` file.

### The Bash Script
```bash
#!/bin/bash

# Find the 'people' file containing the list of suspects
suspects_file=$(find /home/admin -type f -name "people" | head -n 1)

# Extract first and last names, then loop through them
awk '{print $1, $2}' "$suspects_file" | while read -r name; do
  
  # Hash the current name
  hash=$(echo "$name" | md5sum | awk '{print $1}')
  
  # Check if the hash matches our target
  if [ "$hash" = "9bba101c7369f49ca890ea96aa242dd5" ]; then
    echo "$name" > ~/mysolution
    echo "Case solved! The murderer is: $name"
    break
  fi
done
