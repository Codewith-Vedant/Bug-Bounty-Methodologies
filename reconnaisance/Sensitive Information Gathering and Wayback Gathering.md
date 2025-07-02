# Information Gathering
This is an important step in Reconnaisance. It is used to find potential sensitive information which are supposed to be private or of internal use only. These information include but are not limited to:
1. PII of Customers(Full name, Address, Mobile No., Credit Card details, etc.)
2. Employee Information
3. Stakeholder information
4. Documents of a company which is strictly for private or internal use only
5. Database Dumps and backups

In this section we are going to discuss 3 main methods of finding these information: Google Dorking, Wayback Gathering and Publicly exposed S3 buckets

---

## Go Installation (Prerequisite)
Go language is used for a lot of tools, but at the same time it's installation can be tedious. The following steps can easily help you install go language and help in installing any go module as well.
  ```bash
  sudo apt update
  sudo apt upgrade
  #Install Go
  #Go to go language official website and get the latest version link for linux and paste here
  wget https://go.dev/dl/go1.24.4.linux-amd64.tar.gz 
  sudo rm -rf /usr/local/go  # Remove old version (if any)
  sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz
  #Open your ~/.bashrc or ~/.zshrc (if using Zsh) and add:
  export PATH=$PATH:/usr/local/go/bin
  export GOPATH=$HOME/go
  export PATH=$PATH:$GOPATH/bin
  #Apply changes
  source ~/.bashrc #In some distributions it is ~/.zshrc. Check using command "echo $SHELL"
  #Verify Installation
  go version
  #For installing tools via go(Ensure go is installed first)
  echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc #Or ~/.zshrc
  source ~/.bashrc #Or ~/.zshrc
  go install <GO_MODULE_PATH>
  ```

## 1. Google Dorking
Google Dorking is the technique of using advanced search operators to find sensitive data exposed by search engines.

### Manual Dorks
The following some useful google dorks for finding Sensitive Information that should not be exposed to public
| Purpose | Dork |
|--------|------|
| Login pages | `site:example.com inurl:login` |
| Exposed files | `site:example.com ext:pdf OR ext:docx OR ext:xls` |
| Open directories | `site:example.com intitle:"index of"` |
| /.git repositories | `inurl:"/.git " intitle:"index of"` |
| Admin portals | `inurl:/admin intitle:"dashboard"` | 

For a more better list of google dorks, refer to:
1. https://github.com/TakSec/google-dorks-bug-bounty
2. https://medium.com/@paritoshblogs/cool-google-dorks-for-bug-bounty-397b53c27b06

You can use tools like [Bigbountyrecon](https://github.com/Viralmaniar/BigBountyRecon), [Goofuzz](https://github.com/m3n0sd0n4ld/GooFuzz) for automating this method.

## 2. Wayback Gathering
Wayback Machine lets you extract old and forgotten endpoints of a domain—useful for finding outdated pages and legacy bugs.

### waybackurls Command line tool
  ```bash
  #Installation
  go install github.com/tomnomnom/waybackurls@latest
  #Usage
  cat domains.txt | waybackurls > urls
  #Use the unique_subs.txt you generated from Subdomain Enumeration
  ```

💡Pro Tip:
Download the [Google Extension of Wayback Machine](https://chromewebstore.google.com/detail/wayback-machine/fpnmgdkabkmnadcjpehmlllkndpkmiak) in your browser. Whenever you find a 403 Access Denied Page, use this extension to find if it was previously archived in the wayback machine.

## 3. Exposed Public S3 buckets
Many companies store their files and data in s3 buckets. If it is not properly configured, then it can lead to sensitive data leakage.
Sensitive Files found in S3 buckets include database dumps, backup files, sensitive log files, files with customer data or employee data, etc.

### Google Dorks for finding exposed amazon S3 buckets
  ```bash
  site:*.s3.amazonaws.com "example.com"
  (site:*.s3.amazonaws.com OR site:*.s3-external-1.amazonaws.com OR site:*.s3.dualstack.us-east-1.amazonaws.com OR site:*.s3.ap-south-1.amazonaws.com) "example.com"
  #Use keywords in these dorks to find any specific details. Eg: Account No., SSN, etc.
  ```

### Command line tools
  ```bash
  #Amazon S3 buckets
  sudo apt install aws-cli
  aws s3 ls s3://abcd --profile gcs --endpoint-url https://storage.googleapis.com
  #DigitalOcean Spaces bucket
  aws s3 ls s3://<bucket-name>/ --endpoint-url https://<region>.digitaloceanspaces.com --no-sign-request
  #For Google Cloud Storage bucket
  curl https://sdk.cloud.google.com | bash
  exec -l $SHELL
  gcloud init
  gsutil ls gs://<BUCKET_NAME>/
  ```
You can use other CLI tools like [S3 Scanner](https://github.com/sa7mon/S3Scanner), [S3BucketMsconf](https://github.com/Atharv834/S3BucketMisconf) to find exposed or misconfigured S3 buckets.

Installation and Usage of S3BucketMisconf:
  ```bash
  #Install dorks-eye beforehand
  git clone https://github.com/BullsEye0/dorks-eye
  #Install S3BucketMisconf
  git clone https://github.com/Atharv834/S3BucketMisconf.git
  cd S3BucketMisconf
  pip install -r requirements.txt
  #Usage
  python dorks-eye.py #Use dork-eye with a google dork(any of mentioned previously) and store it in a file results.txt
  python updatednews3main.py #Copy and paste the path of results file in it
  ```

💡Pro Tip:
Use the [Google Extension of S3BucketList](https://chromewebstore.google.com/detail/s3bucketlist/anngjobjhcbancaaogmlcffohpmcniki) while manually checking s3 buckets. This helps to tell if that S3 bucket is not properly configured or has public listing enabled.