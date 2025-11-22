# Full Web Recon & Pentesting Workflow (Obsidian Guide)

This guide contains **every step**, **every tool**, **all
explanations**, and **all commands with output saved to files**,
designed for Obsidian.

------------------------------------------------------------------------

## 📌 0. Preparation

### Create project folder

``` bash
mkdir ~/pentest-target
cd ~/pentest-target
```

------------------------------------------------------------------------

## 📌 1. Subdomain Enumeration

### 1.1 Subfinder

``` bash
subfinder -d testphp.vulnweb.com -o subdomains.txt
```

------------------------------------------------------------------------

## 📌 2. DNS + Host Information

``` bash
dig testphp.vulnweb.com ANY > dns.txt
nslookup testphp.vulnweb.com > nslookup.txt
whois testphp.vulnweb.com > whois.txt
```

------------------------------------------------------------------------

## 📌 3. Port Scanning (Nmap)

### Fast scan

``` bash
nmap -T4 -p- testphp.vulnweb.com -oN nmap-ports.txt
```

### Service/version scan

``` bash
nmap -sV -sC testphp.vulnweb.com -oN nmap-services.txt
```

------------------------------------------------------------------------

## 📌 4. Directory Bruteforce (Gobuster)

``` bash
gobuster dir -u https://testphp.vulnweb.com \
    -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
    -o gobuster.txt
```

------------------------------------------------------------------------

## 📌 5. Technologies

``` bash
whatweb https://testphp.vulnweb.com > whatweb.txt
```

------------------------------------------------------------------------

## 📌 6. Crawling & URL Collection

### 6.1 Hakrawler

``` bash
echo "http://testphp.vulnweb.com" | hakrawler -subs -insecure -d 3 -usewayback > hakrawler.txt
```

### 6.2 Waybackurls

``` bash
echo "testphp.vulnweb.com" | waybackurls > wayback.txt
```

### 6.3 GAU

``` bash
echo "testphp.vulnweb.com" | gau > gau.txt
```

### 6.4 Katana

``` bash
katana -u https://testphp.vulnweb.com -o katana.txt
```

### 6.5 Combine all URLs

``` bash
cat hakrawler.txt wayback.txt gau.txt katana.txt | sort -u > all-urls.txt
```

### 6.6 Extract JavaScript files

``` bash
grep -Ei "\.js$" all-urls.txt | sort -u > js-files.txt
```

### 6.7 Extract parameters from URLs

``` bash
grep "=" all-urls.txt > params.txt
```

------------------------------------------------------------------------

## 📌 7. Vulnerability Scanning (Automated)

### 7.1 Nikto

``` bash
nikto -h https://testphp.vulnweb.com -o nikto-report.txt
```

### 7.2 Nmap Vulnerability Scripts

``` bash
nmap -sV --script=vuln -Pn testphp.vulnweb.com -oN nmap-vuln.txt
```

### 7.3 Nmap Web Scripts

``` bash
nmap --script=http-enum,http-methods,http-config-backup \
     -p80,443 testphp.vulnweb.com \
     -oN nmap-web.txt
```

------------------------------------------------------------------------

## 📌 8. Automated Parameter-Based Testing

### 8.1 XSS Testing (Dalfox)

``` bash
dalfox file params.txt --output xss.txt
```

### 8.2 SQL Injection Testing

``` bash
sqlmap -m params.txt --batch --level 3 --risk 2 -o sqlmap-output
```

### 8.3 LFI Testing (FFUF)

``` bash
while read url; do
  ffuf -u "${url}FUZZ" \
      -w /usr/share/wordlists/lfi.txt \
      -o lfi-results.txt
done < params.txt
```

### 8.4 Open Redirect Testing

``` bash
cat params.txt | grep -Ei "redirect|url=" \
    | qsreplace "http://evil.com" \
    | httpx -silent > open-redirect.txt
```

------------------------------------------------------------------------

## 📌 9. Manual Exploitation

### 9.1 XSS

    "><script>alert(1)</script>

### 9.2 SQL Injection

    ?id=1'

### 9.3 LFI

    ?page=../../../../etc/passwd

### 9.4 IDOR

    /userinfo.php?id=1
    /userinfo.php?id=2

------------------------------------------------------------------------

## 📌 10. Final Step: Build Report

Include:

✔ Summary\
✔ All findings\
✔ Screenshots\
✔ Proof of Concept\
✔ Fix recommendations

------------------------------------------------------------------------

## 📌 Why We Collect All This Data?

-   To discover hidden paths\
-   To find URLs with parameters\
-   To detect JS files that expose API endpoints\
-   To map the whole attack surface\
-   To feed automation tools\
-   To speed up manual testing

This is the **complete recon pipeline used by real pentesters and bug
bounty hunters**.

------------------------------------------------------------------------

## ✅ Your workflow is now fully complete.
