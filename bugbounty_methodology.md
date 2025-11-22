# Bug Bounty Toolkit & Methodology (Gentil Security)

## 🛠️ Tools Installation

### **Python3**

``` bash
sudo apt-get update -y
sudo apt-get dist-upgrade -y
sudo apt install -y python3 python3-pip python3.12-venv
python3 --version
pip3 --version
```

### **Golang**

``` bash
sudo apt-get update -y
sudo apt-get dist-upgrade -y
wget https://go.dev/dl/go1.23.3.linux-amd64.tar.gz
rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.3.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
echo 'export PATH=$PATH:~/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
rm -rf go1.23.3.linux-amd64.tar.gz
```

### **CMake**

``` bash
sudo apt-get update -y
sudo apt-get dist-upgrade -y
sudo apt install cmake -y
```

### **Rust**

``` bash
sudo apt-get update -y
sudo apt-get dist-upgrade -y
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
rustc --version
```

------------------------------------------------------------------------

## 🔧 Tools Setup

### **urldedupe**

``` bash
git clone https://github.com/ameenmaali/urldedupe.git
cd urldedupe
cmake CMakeLists.txt
make
sudo cp urldedupe /usr/local/bin/
cd ../
urldedupe -h
```

### **Arjun**

``` bash
git clone https://github.com/s0md3v/Arjun.git
python3 -m venv arjun-env
source arjun-env/bin/activate
cd Arjun
pip3 install .
sudo cp ~/arjun-env/bin/arjun /usr/local/bin/
deactivate
```

### **Waymore**

``` bash
git clone https://github.com/xnl-h4ck3r/waymore.git
python3 -m venv waymore-env
source waymore-env/bin/activate
cd waymore
pip3 install .
pip3 install -r requirements.txt
sudo cp ~/waymore-env/bin/waymore /usr/local/bin/
deactivate
```

### **Sublist3r**

``` bash
git clone https://github.com/aboul3la/Sublist3r.git
python3 -m venv sublist3r-env
source sublist3r-env/bin/activate
cd Sublist3r
pip3 install .
pip3 install -r requirements.txt
sudo cp ~/sublist3r-env/bin/sublist3r /usr/local/bin/
deactivate
```

### **Dirsearch**

``` bash
git clone https://github.com/maurosoria/dirsearch.git --depth 1
python3 -m venv dirsearch-env
source dirsearch-env/bin/activate
cd dirsearch
pip3 install .
pip3 install -r requirements.txt
sudo cp ~/dirsearch-env/bin/dirsearch /usr/local/bin/
deactivate
```

### **subExtreme**

``` bash
sudo apt install pkg-config libssl-dev -y
git clone https://github.com/ahmedhamdy0x/subextreme.git
cd subextreme
cargo build --release
sudo cp target/release/subextreme /usr/local/bin/
sudo chmod +x /usr/local/bin/subextreme
```

### **httpx**

``` bash
sudo rm -f /usr/bin/httpx && sudo apt remove httpx -y
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
```

### **Crawley**

``` bash
mkdir crawley
cd crawley
wget https://github.com/s0rg/crawley/releases/download/v1.7.10/crawley_v1.7.10_linux_x86_64.tar.gz
tar -xvzf crawley_v1.7.10_linux_x86_64.tar.gz
sudo cp crawley /usr/local/bin/
```

### **PassURLs**

``` bash
git clone https://github.com/ahmedhamdy0x/passurls.git
python3 -m venv passurls-env
source passurls-env/bin/activate
cd passurls
pip3 install .
pip3 install -r requirements.txt
sudo cp ~/passurls-env/bin/passurls /usr/local/bin/
```

### **Dalfox**

``` bash
go install github.com/hahwul/dalfox/v2@latest
```

### **SecLists**

``` bash
git clone https://github.com/danielmiessler/SecLists.git
```

------------------------------------------------------------------------

# 🧭 Gentil Security Hacking Methodology

## 📁 Initial Setup

``` bash
mkdir bugbounty
cd bugbounty
mkdir <target>
cd <target>
```

------------------------------------------------------------------------

## 🌐 Subdomain Enumeration

``` bash
sublist3r -d example.com -b -t 50 -v -o sublist3r.txt
subextreme -w ~/Seclists/Discovery/DNS/nokovo_subdomains.txt -d example.com -c 100 -o subextreme.txt
cat sublist3r.txt subextreme.txt | urldedude -s > subdomains.txt
```

------------------------------------------------------------------------

## ✔️ Filter Valid Subdomains

``` bash
httpx -l subdomains.txt -o valid-subs.txt -t 60 -random-agent -mc 200
```

------------------------------------------------------------------------

## 🕸️ Extract URLs (Passive)

``` bash
cat valid-subs.txt | waymore -mode B -oU waymoreurls.txt -p 5 -mc 200
cat waymoreurls.txt | urldedude -s > uniq-urls.txt
```

------------------------------------------------------------------------

## 🔄 Send URLs to Burpsuite

``` bash
passurls -p 127.0.0.1:8080 -l uniq-urls.txt
```

------------------------------------------------------------------------

## 🗂️ Directory Enumeration

``` bash
dirsearch -u <target> -e php,html,log,sql,zip,xml,json -w ~/Seclists/Discovery/Web-Content/combined-words.txt -o hidden-files.txt
```

------------------------------------------------------------------------

## 🕷️ Crawley Deep Crawling

``` bash
export http_proxy="http://127.0.0.1:8080"
export https_proxy="http://127.0.0.1:8080"
export ALL_PROXY="http://127.0.0.1:8080"

crawly -headless -delay 10ms -depth -1 -subdomains -all -timeout 30s -user-agent "Mozilla/5.0" -dirs dirsearch -raw crawly.txt
```

------------------------------------------------------------------------

## 📌 Extract Parameters

``` bash
nano burp-results.txt
grep -E '.*=' burp-results.txt > burp-params.txt
cat burp-params.txt | urldedude -s > parameters.txt
```

------------------------------------------------------------------------

## 🔍 Find Hidden Parameters (xfunj)

``` bash
xfunj -l hidden-files.txt -t 10 -c 300 -T 30 -d 10 -p 127.0.0.1:8080 -m POST -oT hidden-params1.txt
xfunj -l burp-results.txt -t 10 -c 300 -T 30 -d 10 -p 127.0.0.1:8080 -m POST -oT hidden-params2.txt
```

------------------------------------------------------------------------

## ⚡ XSS Scanning (Dalfox)

``` bash
dalfox file parameters.txt -waf-evasion -user-agent "Mozilla/5.0" -timeout 30 -proxy "http://127.0.0.1:8080" -x "(img src=x onerror=alert(1))" --deep-domxss -o dalfox.txt
```
------------------------------------------------------------------------
# thanks
