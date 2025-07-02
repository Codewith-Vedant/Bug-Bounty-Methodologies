# Parameter Enumeration
Parameter Enumeration is the process of discovering GET/POST parameters that a web application accepts — including hidden, undocumented, or deprecated ones. These parameters can be potential vectors for Injection attacks (SQLi, XSS, etc.), Bypassing controls (e.g., admin=true), Accessing hidden functionality (e.g., debug=true) and API misbehavior (e.g., id=../../etc/passwd).

---

## Tools & Techniques for Parameter Enumeration

### 1. Crawling + Passive Discovery
Identify parameters through crawling site URLs and forms.
Tools like Burp Suite(Proxy + Spider), [hakrawler](https://github.com/hakluke/hakrawler) and [gospider](https://github.com/jaeles-project/gospider) come to be extremely handy for crawling
  ```bash
  gospider -s https://example.com -d 2 -c 50 | grep '='
  ```
[Broken Link Checker](https://github.com/stevenvachon/broken-link-checker) can also be used as an alternative tool for website crawling(not parameter discovery). It's main purpose is to check for broken links in the website. 
  ```bash
  blc -rof --debug https://example.com
  ```

### 2. Using GAU
  ```bash
  gau example.com | grep '=' > params.txt
  ```

### 3. Using Arjun
Active Parameter fuzzing by brute-forcing parameter names to discover hidden or undocumented ones.
[Arjun](https://github.com/s0md3v/Arjun) is designed specifically for discovering GET/POST parameters and Uses a large wordlist and detects changes in server responses.
  ```bash
  #For GET
  python3 arjun.py -u https://example.com/search -m GET
  #For POST
  python3 arjun.py -u https://example.com/api/v1/login -m POST
  ```

💡Pro Tip:
Use [Linkfinder](https://github.com/GerbenJavado/LinkFinder) to analyze js files and find any hidden parameters(for APIs, form submissions, or debugging). This tool was specifically made to do endpoint and parameter enumeration.
  ```bash
  python3 linkfinder.py -i https://example.com/assets/app.js -o cli
  ```

### 4. Using Paramspider
  ```bash
  python3 paramspider.py --domain example.com --exclude woff,css,png,svg --level high
  ```

### 5. Using ffuf
  ```bash
  ffuf -u https://example.com/page.php?FUZZ=test -w params.txt -fs 0
  ```

💡Pro Tip:
For multiple urls, use [qsreplace](https://github.com/tomnomnom/qsreplace) and then pass the output to ffuf.
  ```bash
  cat urls.txt | qsreplace "FUZZ" > testable_urls.txt
  ffuf -u FUZZ -w payloads.txt -request fuzzable_urls.txt
  ```

----
## Optimized Pipeline Script for Parameter Enumeration
  ```bash
  #!/bin/bash
  
  DOMAIN=$1
  OUTPUT_DIR="enum_$DOMAIN"
  mkdir -p $OUTPUT_DIR
  
  echo "[*] Running gau..."
  gau $DOMAIN | tee $OUTPUT_DIR/gau.txt | grep '=' > $OUTPUT_DIR/gau_params.txt
  echo "[*] Running waybackurls..."
  echo $DOMAIN | waybackurls | tee $OUTPUT_DIR/wayback.txt | grep '=' > $OUTPUT_DIR/wayback_params.txt
  
  echo "[*] Running ParamSpider..."
  python3 paramspider/paramspider.py --domain $DOMAIN --output $OUTPUT_DIR/paramspider.txt
  
  echo "[*] Extracting JS files from gau..."
  cat $OUTPUT_DIR/gau.txt | grep ".js" | uniq > $OUTPUT_DIR/js_urls.txt
  
  echo "[*] Running LinkFinder on JS files..."
  while read jsurl; do
     python3 LinkFinder/linkfinder.py -i $jsurl -o cli >> $OUTPUT_DIR/linkfinder_params.txt
  done < $OUTPUT_DIR/js_urls.txt

  echo "[*] Merging and extracting unique parameter names..."
  cat $OUTPUT_DIR/*params.txt | grep '=' | grep -oP '\b[a-zA-Z0-9_\-]{1,30}(?==)' | sort -u > $OUTPUT_DIR/param_names.txt

  echo "[*] Active Fuzzing with arjun..."
  python3 Arjun/arjun.py -u https://$DOMAIN -oT $OUTPUT_DIR/arjun_output.txt
  
  echo "[*] Active Brute-force with ffuf on common param names..."
  ffuf -u "https://$DOMAIN/test.php?FUZZ=test" -w $OUTPUT_DIR/param_names.txt -fs 0 -o $OUTPUT_DIR/ffuf_results.json
  
  echo "[*] Done! Results saved in $OUTPUT_DIR"
  ```

Usage:
  ```bash
  chmod +x param_enum.sh
  ./param_enum.sh example.com
  ```

Output Files:
1. gau.txt, wayback.txt: Full archive URLs
2. gau_params.txt, etc.: URLs with parameters
3. param_names.txt: Unique parameter names
4. arjun_output.txt: Discovered parameters (via brute-force)
5. ffuf_results.json: Live parameters with 200 OK responses