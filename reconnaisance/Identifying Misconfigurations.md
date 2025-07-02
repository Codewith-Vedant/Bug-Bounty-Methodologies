# Identifying Misconfigurations
In bug bounty hunting, identifying misconfigurations involves finding improperly set security controls that expose the application or infrastructure to attacks. These misconfigurations are often low-hanging fruits that can lead to high-impact exploits when chained with other vulnerabilities.

---

## CORS Misconfigurations
CORS (Cross-Origin Resource Sharing) controls which domains can access a web app’s resources. A misconfigured CORS policy can allow attackers to read sensitive data by making cross-origin requests.

### Using Corsy
  ```bash
  python3 corsy.py -u https://example.com
  ```

### Using nuclei
  ```bash
  nuclei -u https://example.com -t misconfiguration/cors.yaml
  ```

### Using curl
  ```bash
  curl -H "Origin: https://evil.com" -I https://example.com
  ```

💡Pro Tip:
If you see the origin being reflected, test with JavaScript in the browser console to try an XHR cross-origin request. Combine with cookies if credentials=true.

## Insecure HTTP Methods
Insecure methods like PUT, DELETE, or TRACE expose functionality that attackers can misuse for file uploads, XST, or data deletion.

### Using http-methods
  ```bash
  #Installation
  git clone https://github.com/ShutdownRepo/httpmethods
  cd httpmethods
  python3 setup.py install
  #Usage
  http-methods -u https://example.com
  ```

### Using Nmap
  ```bash
  nmap -p80,443 --script http-methods https://example.com
  ```

### Using Curl
  ```bash
  curl -X OPTIONS https://example.com -i
  curl -X PUT https://example.com/shell.php --data-binary @shell.php
  ```

💡 Pro Tip:
If PUT is allowed, try uploading a web shell to a path like /uploads/ or root. Then check if the file is publicly accessible.

## Verbose Error Messages
Exposed stack traces and debug messages can reveal backend logic, directory paths, or database queries — invaluable for crafting targeted payloads.

### Using Nuclei
  ```bash
  nuclei -u https://target.com -t exposures/stack-trace.yaml
  ```

### Using Nikito
  ```bash
  nikto -h https://target.com
  ```

💡 Pro Tip:
Change input types or headers unexpectedly (e.g., send XML when expecting JSON) — this often triggers unhandled exceptions.

## Scope-wide scanning with Nuclei 
  ```bash
  nuclei -l urls.txt -t misconfiguration/
  ```