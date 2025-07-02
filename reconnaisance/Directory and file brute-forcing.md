# Directory and file brute-forcing
Directory and file brute-forcing is a key part of web reconnaissance and vulnerability assessment. It helps you discover Hidden admin panels, Backup files (.zip, .bak), Development directories (/test/, /staging/), Misconfigured APIs or uploads folders, etc.

---

## Common tools for brute-forcing

### Using dirb
  ```bash
  dirb https://example.com <wordlist_file_path>
  ```
By default dirb uses the path /usr/share/wordlists/dirb/common.txt

### Using dirsearch
  ```bash
  dirsearch -u https://example.com <wordlist1.txt,wordlist2.txt>
  ```

### Using gobuster
  ```bash
  gobuster dir -u http://1example.com -w /usr/share/seclists/Discovery/Web-Content/common.txt -b <Status_You_Do_Not_Need>
  ```
`-b` extension is used to give negative status codes.

### Using ffuf (Fuzz Faster U Fool)
  ```bash
  ffuf -u https://example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -mc all
  ffuf -u https://example.com/FUZZ.php -w wordlist.txt -mc 200 #For specific file extensions
  ```

💡Pro Tip:
If you want to fuzz the input you can give in a website endpoint(like https://bank.com/profile?user=1234), you can use ffuf in the following way:
  ```bash
  ffuf -u "http://example.com/logs/view?name=FUZZ" -w log_names.txt -mc all
  ```

## 🔎 Recommended Wordlists
|Tool |	Path|
|--------|------|
|Dirb |	/usr/share/wordlists/dirb/|
|SecLists |	/usr/share/seclists/Discovery/Web-Content/|
|Bug Bounty Wordlists | [Link Here](https://github.com/Karanxa/Bug-Bounty-Wordlists)|

💡Pro Tip:
Use [backup-gen](https://github.com/Nishantbhagat57/backup-gen) to generate wordlists for backup files and bruteforce them for any website with ffuf or gobuster.