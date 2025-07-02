# Enumeration
Enumeration in bug bounty hunting is the process of actively gathering detailed information about the discovered assets to uncover attack surfaces. It bridges passive recon and active vulnerability discovery.

---

## Port and Service Scanning
Identify open ports and services running on them.

### Using Naabu
Prerequisite: Go should be installed properly.
  ```bash
  #Installation
  go install -v github.com/projectdiscovery/naabu/v2/cmd/naabu@latest
  #Usage
  naabu -iL live_hosts.txt -p - -o ports.txt
  ```

### Using nmap
  ```bash
  nmap -iL live_hosts.txt -p- -sV -T4 -oN nmap_output.txt
  ```

### Using rustscan
  ```bash
  rustscan -a <target> --ulimit 5000 -- -A -sV -oN rustscan_output.txt
  ```

💡Pro Tip:
For faster scanning of all 65536 ports, use rustscan instead of nmap.

### Using massdns
  ```bash
  masscan -iL live_hosts.txt -p1-65535 --rate=10000 -oG masscan_output.gnmap
  ```

💡Pro Tip:
Port scanning a CDN (like Cloudflare) won't reveal real services — scan resolved origin IPs.

## Default Credentials and Weak Auth Checks
Identify login portals that use default, weak, or guessable credentials.

### Using Hydra (Login Brute-forcing)
  ```bash
  hydra -l admin -P passwords.txt example.com http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"
  ```

### Nmap NSE Scripts (Default Passwords)
  ```bash
  nmap -p21,22,23,80,443,8080,3306,161 \
  -sV \
  --script ftp-brute,ssh-brute,telnet-brute,http-brute,mysql-brute,snmp-brute \
  --script-args userdb=users.txt,passdb=pass.txt,brute.firstonly=true \
  -T4 -oN default_creds_scan.txt <target>
  ```
This is useful for checking other ports like FTP, telnet as well.

### Using Medusa (Parallel Login Cracker)
It is faster compared to hydra on larger login lists.
  ```bash
  medusa -h example.com -U users.txt -P passwords.txt -M http -m DIR:/login -m FORM:username=^USER^&password=^PASS^
  ```

📂 Wordlists:
1. /usr/share/wordlists/rockyou.txt
2. /usr/share/SecLists/Passwords/Default-Credentials/