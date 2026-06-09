# Scenario: "Taipei": Come a-knocking

**Level:** Easy  
**Tags:** `security`, `networking`, `port-knocking`, `bash`, `nmap`  

## 📝 Problem Description
A local web server on port `:80` is protected by a port-knocking firewall rule. Accessing `curl localhost` initially returns a decoy message (`Who is there?`) or is blocked. To unlock the real page, a specific closed port must receive a packet connection attempt ("knock") to trigger the background daemon (`knockd`) into opening port 80. The configuration file `/etc/knockd.conf` is locked down with permission restrictions.

## 💡 The Solution Strategy
When manual inspection of the firewall configuration is blocked by file permissions, network scanning or automated scripting can bypass the restriction by scanning ranges or hitting ports sequentially until the trigger condition is satisfied.

### Approach 1: Rapid Portfolio Scanning (Tricking the Firewall)
Running an all-ports scan using `nmap` naturally hits the secret port during its discovery process, inadvertently triggering the `knockd` daemon to drop the firewall restriction on port 80.

```bash
# Scan all 65,535 ports at lightning speed
nmap -p- localhost

# Verify the web server has been unlocked
curl localhost

### Approach 2: Programmatic Brute Force Loop
If explicit scanning tools are unavailable, a standard shell loop can sequentially ping ports using the system's `knock` utility while validating the state of the grading script.

```bash
for port in {1..9000}; do
  knock localhost $port
  if [ "$(./check.sh)" = "OK" ]; then
    echo "Success! The secret port is $port"
    curl localhost
    break
  fi
done
