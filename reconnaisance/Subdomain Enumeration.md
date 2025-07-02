Reconnaisance is the first step of Bug Bounty hunting and also the most important step in bug bounty. It is used to map the attack surface, find sensitive exposed files and vulnerable endpoints.

It would be covered in 7 parts:
1. Subdomain enumeration and finding live hosts
2. Sensitive Information Disclosure using google dorking and wayback
3. DNS & IP Recon
4. Technology Stack and CMS Fingerprinting of live hosts
5. Directory and file brute-forcing
6. Parameter Enumeration 
7. Identifying Misconfigurations

---

# Subdomain Enumeration
Subdomain Enumeration can be done in 2 ways - Active and Passive. It was always recommended to start with passive subdomain enumeration and then follow that with active subdomain enumeration.
## Using Subfinder and httpx(Passive only)
  ```bash
  subfinder -d example.com -silent > subs.txt
  httpx -silent -status-code -title -tech-detect < subs.txt > live_hosts.txt
  ```

## Full Passive Subdomain Enumeration
  ```bash
  subfinder -d example.com -all -silent -o subfinder.txt
  assetfinder --subs-only example.com > assetfinder.txt
  amass enum -passive -d example.com -o amass_passive.txt
  findomain -t example.com -u findomain.txt
  curl -s "https://crt.sh/?q=%.example.com&output=json" | jq -r '.[].name_value' | sort -u > crtsh.txt
  curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u | tee url.txt
  ```

## Full Active Subdomain Enumeration
  ```bash
  amass enum -active -brute -d example.com -o amass_active.txt
  shuffledns -d example.com -w ~/wordlists/subdomains.txt -r ~/resolvers.txt -o shuffledns.txt
  massdns -r ~/resolvers.txt -t A -o S ~/wordlists/subdomains.txt > massdns_output.txt
  ```

💡Pro Tip:
Make a folder seperate to store your work and structure your work in it for better organisation. This helps to keep your work look clean. After doing the above full passive and active subdomain enumeration, use the folllowing command to get unique subdomains:
  ```bash
  cat *.txt | sort -u > all_unique_subs.txt
  ```

## All subdomains(fast + deep with Active and Passive Enumeration)
  ```bash
  subfinder -d example.com > subdomains.txt
  assetfinder --subs-only example.com >> subdomains.txt
  amass enum -passive -d example.com >> subdomains.txt
  sort -u subdomains.txt -o unique_subs.txt
  dnsx -l unique_subs.txt -silent -o live_subs.txt
  ```

## Using Shrewedeye
  ```bash
  wget https://shrewdeye.app/domains/<domain_name>.txt 
  ```

## Using 3rd Party services and API Keys(Optional)
You can use 3rd party services and APIs as well to add more subdomains in it
  ```bash
  # Virustotal
  curl -s --request GET \
  --url "https://www.virustotal.com/api/v3/domains/example.com/subdomains?limit=40" \
  --header "x-apikey: <VT_API_KEY>" | jq -r '.data[].id'
  # Shodan
  shodan search --fields hostnames,ip_str "hostname:example.com"
  #Threatcrowd
  curl -s "https://www.threatcrowd.org/searchApi/v2/domain/report/?domain=example.com" \
  | jq -r '.subdomains[]'
  #AlienVault OTX
  curl -s "https://otx.alienvault.com/api/v1/indicators/domain/example.com/passive_dns" \
  | jq -r '.passive_dns[] | .hostname'
  #SecurityTrails
  curl -s "https://api.securitytrails.com/v1/domain/example.com/subdomains?apikey=<API_KEY>" \
  | jq -r '.subdomains[]' | sed 's/^/example.com./'
  #Censys
  censys subdomains example.com --api-id <ID> --secret <SECRET>
  #Spyse
  curl -X GET "https://api.spyse.com/v4/data/domain/subdomain?domain=example.com" \
  -H "Authorization: Bearer <API_KEY>"
  #BinaryEdge
  curl -s "https://api.binaryedge.io/v1/domain/example.com/subdomains" \
  -H "X-Key: <API_KEY>" | jq
  ```

Web UI tools like [DNSDumpster](https://dnsdumpster.com/) can also be used for the same.

---

# Live host detection
For live host detection, you can use tools like httpx, dnsx, gau

## Using gau
getallurls (gau) fetches known URLs from AlienVault's Open Threat Exchange, the Wayback Machine, Common Crawl, and URLScan for any given domain
  ```bash
  #Installation
  go install github.com/lc/gau/v2/cmd/gau@latest
  #Usage
  echo "example.com" | gau > gau.txt
  ```
Check it's [Github page](https://github.com/lc/gau) for more details.

## Using httpx
  ```bash
  httpx -l resolved_subs.txt -ports 80,443,8080,8443 -title -status-code
  ```
