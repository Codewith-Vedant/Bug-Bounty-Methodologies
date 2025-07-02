# JS File Analysis
JavaScript files often contain rich information that can reveal sensitive data, API endpoints, and internal logic. Proper analysis can lead to severe findings like unauthorized access, IDORs, or exposed secrets.

---

## Finding JavaScript Files

### Using gau
  ```bash
  gau example.com | grep -Ei '\.js$' | sort -u > jsfiles.txt
  ```

### Using hakrawler
  ```bash
  hakrawler -url https://example.com -subs -js -depth 3 > jsfiles_live.txt
  ```

### Using waybackurls
  ```bash
  waybackurls example.com | grep -Ei '\.js$' | sort -u >> jsfiles.txt
  ```

## Exposed API Keys and Secrets
 
### Using Secretfinder
  ```bash
  cat js.txt | while read url; do
    python3 SecretFinder.py -i "$url" -o cli >> secrets_found.txt
  done
  ```

### Using gf
  ```bash
  cat js.txt | xargs -n1 curl -s | gf aws-keys
  ```

### Using Nuclei
  ```bash
  nuclei -l js.txt -t ~/nuclei-templates/exposures/
  ```

💡Pro Tip:
Use [LeakFox](https://github.com/Codewith-Vedant/LeakFox) to automate the testing of valid API keys. It can test across 70+ different API Keys and can take multiple input via file as well. It is available in both Windows and Linux Versions.

## Enumerating API Endpoints

### Using Linkfinder
  ```bash
  cat js.txt | while read url; do
    python3 linkfinder.py -i "$url" -o cli >> endpoints.txt
  done
  ```

### Using Manual grep
  ```bash
  cat js.txt | xargs -n1 curl -s | grep -oP "https?:\/\/[^\s\"']+" | sort -u >> endpoints.txt
  ```

💡Pro Tip:
Check methods: Is it only GET/POST, or PUT/DELETE too?

## Analyzing Auth & Token Flow
Many websites store their auth tokens(JWT, etc.) directly in js files.

### Using jwt_tool
  ```bash
  # Grep tokens in JS
  cat js.txt | xargs -n1 curl -s | grep -Ei "Bearer\s[a-zA-Z0-9\-_\.]+" >> tokens.txt
  # Decode JWT tokens
  cat tokens.txt | awk '{print $2}' | while read token; do
    jwt_tool "$token"
  done
  ```

### Manual testing using jwt.io
Once you find a jwt token in a js file, go to [jwt.io](https://jwt.io/) and paste it there to decode it. Some auth tokens have payloads like `"admin": False`. Manually intercept using burpsuite and change the jwt token to see if you get admin access.

----

## Automated Full Workflow (1-liner script)
  ```bash
  # One-liner to parse all secrets and endpoints
  cat js.txt | while read url; do
    
    echo "[*] Analyzing $url"
    curl -s "$url" -o temp.js
    
    echo "-> Secrets:" >> summary.txt
    python3 SecretFinder.py -i "$url" -o cli >> summary.txt
    
    echo "-> Endpoints:" >> summary.txt
    python3 linkfinder.py -i "$url" -o cli >> summary.txt
    
    echo "-> Raw JS Saved as: $(basename $url)" >> summary.txt
  done
  ```
