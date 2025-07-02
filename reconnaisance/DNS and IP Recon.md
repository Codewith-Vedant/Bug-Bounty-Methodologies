# DNS and IP Recon
DNS and IP Reconnaissance refers to the process of gathering information about domain names, their corresponding IP addresses, associated subdomains, DNS records, and network infrastructure. This is a critical phase in ethical hacking, penetration testing, and bug bounty hunting.

In this section we are gona talk about 3 widely known tactics: DNS Enumeration, IP Enumeration and Origin IP Discovery.

---

## DNS Enumeration
The goal in this step is to gather domain-related information like subdomains, DNS records, mail servers, nameservers, etc.

### Using dig
  ```bash
  dig example.com                 # Get A record
  dig MX example.com             # Mail servers
  dig NS example.com             # Nameservers
  dig TXT example.com            # SPF/DKIM/DMARC info
  dig ANY example.com            # All available records
  dig AXFR example.com @ns1.example.com  # Zone transfer (if allowed)
  ```

### Using nslookup
  ```bash
  nslookup example.com
  nslookup -type=MX example.com
  nslookup -type=NS example.com
  ```

### Using Host
  ```bash
  host example.com
  host -t MX example.com
  host -t NS example.com
  ```

### Certificate Transperancy Logs
  ```bash
  curl -s "https://crt.sh/?q=%.example.com&output=json" | jq -r '.[].name_value' | sort -u
  ```

## IP Enumeration
The goal in this step is to map domains and subdomains to IP addresses, analyze IP ranges, and check service exposure.

### Get IP of domain/subdomain
  ```bash
  dig +short example.com
  host sub.example.com
  nslookup sub.example.com
  ```

### DNS Probing for Subdomains
  ```bash
  subfinder -d example.com | dnsx -a -resp
  ```

### Reverse DNS Lookup
  ```bash
  host 192.0.2.1
  dig -x 192.0.2.1
  ```

### IP & ASN Info
  ```bash
  whois 192.0.2.1
  curl ipinfo.io/192.0.2.1
  curl -s "https://bgp.he.net/ip/192.0.2.1"
  ```

## Origin IP Discovery
The goal in this step is to reveal the real server IP address behind CDNs like Cloudflare, Akamai, etc. CDNs hide the origin IP to protect from DDoS and direct attacks. Finding the origin IP allows us to inject our payloads directly to the server, without any need to bypass the CDN.

### Find Misconfigured Subdomains
  ```bash
  subfinder -d example.com | dnsx -a -resp | grep -vE 'Cloudflare|Akamai|Fastly'
  ```

### Shodan Search
  ```bash
  # In Shodan search bar:
  ssl:"example.com"
  http.favicon.hash:12345678     # If known
  http.title:"Welcome to example"
  ```

💡Pro Tip:
Use the [Favicon Finder](https://favicons.teamtailor-cdn.com/) to get the favicon.ico file of a website. Use the url of this .ico file to get the favicon hash by using the [Favicon hash generator](https://favicon-hash.kmsec.uk/). From the favicon hash generator, you can directly launch the Censys, Virustotal and Shodan search.

### Censys Search
  ```bash
  services.tls.certificates.leaf_data.names: "example.com"
  services.http.response.favicons.md5_hash: <MD5_Hash_of_Favicon>
  ```

### Using AlienVault
  ```bash
  domain="example.com"
  curl -s "https://otx.alienvault.com/api/v1/indicators/hostname/${domain}/url_list?limit=500&page=1" | jq -r '.url_list[]?.result?.urlworker?.ip // empty' | grep -Eo '([0-9]{1,3}\.){3}[0-9]{1,3}' | sort -u > temp_ips.txt

### SPF and DMARC Records
  ```bash
  dig TXT example.com
  #Check for ip4: and ip6: values in SPF records—sometimes reveals mail server or origin IP
  ```

### Historical DNS Data
Use tools like securitytrails.com, viewdns.info and dnsdumpster.com show past A records before CDN protection was applied.

###  LeakIX / BinaryEdge / ZoomEye
Use [LeakIX](https://leakix.net/) Search for leaked origin IPs of domains.

Once an IP is suspected, you can test it by directly opening it in the browser.
  ```bash
  http://<Suspected_IP>
  ```

Use tools like [Cloudfail](https://github.com/m0rtem/CloudFail) to get origin IP of a target protected by Cloudflare.