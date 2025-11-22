---
aliases: [nmap, "Nmap quick reference"]
tags: [pentest, nmap, networking]
---

# Nmap Quick Notes

Short, Obsidian-friendly reference for common `nmap` scan types and commands.

> **Quick reminder:** always have explicit permission before scanning any host or network.

## Scans & Commands

### 1. Ping scan — `-sn`
- Purpose: Quickly discover **live hosts** on a network without scanning for open ports.
- Command:
```bash
nmap -sn <target>
```
- Notes: Useful as a first step to map which IPs are up.
- **Deprecated alias:** `-sP` was the older name for the ping scan in previous `nmap` versions. It behaves the same as `-sn` but has been deprecated — use `-sn` in modern usage.

### 2. SYN scan (half-open) — `-sS`
- Purpose: Default and most popular TCP scan. Sends SYN and inspects replies (stealthier than full connect).
- Command:
```bash
nmap -sS <target>
```
- Notes: Often faster and less noisy than a full TCP connect scan; requires raw socket privileges on some systems.

### 3. UDP scan — `-sU`
- Purpose: Checks for open UDP ports.
- Command:
```bash
nmap -sU <target>
```
- Notes: UDP scans are slower and can produce false negatives due to lack of replies; combine with version detection for better results.

### 4. Service/version detection — `-sV`
- Purpose: Determines the **service name** and **version** running on open ports.
- Command:
```bash
nmap -sV <target>
```
- Notes: Useful to identify vulnerable versions; pair with `-p` to target specific ports.
- **Port selection:** `-p` lets you scan specific ports or ranges. Examples:
  - Scan specific ports:
```bash
nmap -p 22,80,443 <target>
```
  - Scan port range:
```bash
nmap -p 1-65535 <target>
```
  - Scan top N common ports:
```bash
nmap --top-ports 100 <target>
```
    - Purpose: Scan the specified number of the most commonly used ports.
    - Notes: Faster than scanning all ports and often catches the services most likely to be running.
    - Combined example:
```bash
nmap --top-ports 100 -sV <target>
```
    - **Comparison:** `--top-ports` is faster but might miss uncommon ports; `-p 1-65535` is comprehensive but slower.

### 5. Operating system detection — `-O`
- Purpose: Attempts to identify the target's **operating system**.
- Command:
```bash
nmap -O <target>
```
- Notes: Requires enough open/closed probes and often elevated privileges; results are probabilistic (not guaranteed).

### 6. Aggressive scan — `-A`
- Purpose: Combines multiple techniques (OS detection, version detection, script scanning, traceroute) into one run.
- Command:
```bash
nmap -A <target>
```
- Notes: Very noisy and more likely to be detected by IDS/IPS — use only with permission.

## Example: Specific command added by user

**Command (as provided):**

```bash
nmap -p 80 -Pn -sV juice-shop.herokuapp.com
```

**Word-by-word explanation:**
- `nmap` — The program/utility name (Network Mapper).
- `-p 80` — `-p` specifies port selection. `80` is the port number to scan (standard HTTP). You can list multiple ports with commas (e.g., `-p 22,80,443`) or a range (`-p 1-1000`).
- `-Pn` — Treat the host as **up** ("no ping"). Skips host discovery (ICMP/ARP/etc) and proceeds to probe ports directly. Useful when ping/ICMP is blocked or filtered, or when scanning hosts behind firewalls.
- `-sV` — Service/version detection. Probes open ports to identify the service name and version (e.g., `nginx 1.18.0`). Helpful to fingerprint software and find version-specific vulnerabilities.
- `juice-shop.herokuapp.com` — The target hostname (nmap resolves it to an IP or set of IPs before scanning).

**Notes & practical tips:**
- This command scans **only TCP port 80** on the resolved IP(s) and attempts to detect the service/version while skipping initial host discovery.
- Because it skips discovery (`-Pn`), you may see slower port probes if the host is actually down (nmap will try and wait for timeouts).
- Running as root is **not required** for this specific command, but some advanced scans (e.g., `-sS`, OS detection `-O`) can require elevated privileges and may give more accurate results.
- Always ensure you have permission to scan the target. Scanning domains hosted on shared platforms (like Heroku) may affect other tenants or violate provider policies.

## Common variants / tips
- Combine scans:
```bash
nmap -sS -sV -p 1-1000 <target>
```
- Run as root / with privileges when required (e.g., `-sS`, `-O`).

## Common variants / tips (ports & top ports)
- Scan specific ports:
```bash
nmap -p 22,80,443 <target>
```
- Scan a range:
```bash
nmap -p 1-65535 <target>
```
- Use the most common ports quickly:
```bash
nmap --top-ports <number> <target>
```
  - Purpose: Scan the specified number of the most commonly used ports (e.g., `--top-ports 100` scans the 100 ports that appear most often in services).
  - Notes: Faster than scanning all ports and often catches the services most likely to be running. Can be combined with other flags like `-sV` for version detection (e.g., `nmap --top-ports 100 -sV <target>`).

## Curated web NSE scripts (recommended)
A short, focused list of web-related NSE scripts pulled from your scripts directory — good for web-app assessments.

- `http-title` — grabs the page title (fast reconnaissance).
- `http-headers` — shows HTTP response headers (security-related headers, server banner).
- `http-robots.txt` — fetches `robots.txt` (information disclosure).
- `http-enum` — enumerates common web directories and files.
- `http-vuln-cve2017-5638` — example vuln check (specific CVE) — **use only with permission**.
- `http-sql-injection` — tests for SQL injection (intrusive).
- `http-fileupload-exploiter` — checks for file upload vulnerabilities (intrusive).
- `http-wordpress-enum` — enumerates WordPress items (users, plugins).
- `http-waf-detect` — tries to detect presence of a Web Application Firewall.
- `http-sitemap-generator` — generates possible sitemap entries.

If you want I can trim/add scripts to this list according to your testing goals.

## Safe vs Intrusive scripts (guideline)
- **Safe / low-impact** (good for initial recon): `http-title`, `http-headers`, `http-robots.txt`, `http-enum`, `http-vhosts`, `http-favicon`.
- **Moderate** (may be noisy or stress the app): `http-enum` (deep), `http-wordpress-enum`, `http-sitemap-generator`.
- **Intrusive / high-risk** (only with explicit authorization and during a controlled test window): `vuln` category scripts, `http-sql-injection`, `http-fileupload-exploiter`, `http-vuln-*` scripts. These can alter application state or trigger alarms.

## Ready-to-run examples (copy/paste)
- Quick HTTP recon (safe):
```bash
nmap -p 80 --script "http-title,http-headers,http-robots.txt" -oA juice_http_recon juice-shop.herokuapp.com
```
- Version detection + common web scripts:
```bash
sudo nmap -sV --script "http-headers,http-enum,http-waf-detect" --top-ports 100 -oA juice_web_checks juice-shop.herokuapp.com
```
- Intrusive vulnerability check (ONLY WITH PERMISSION):
```bash
nmap -sV -p80 --script=vulnrs.nse -oA juice_intrusive_checks juice-shop.herokuapp.com
```
- Save output in XML for reports:
```bash
nmap -sS -sV --top-ports 100 -oX juice_report.xml juice-shop.herokuapp.com
```

## Selected scripts (short pasted list)
- http-title.nse
- http-headers.nse
- http-robots.txt.nse
- http-enum.nse
- http-sql-injection.nse
- http-fileupload-exploiter.nse
- http-waf-detect.nse
- http-wordpress-enum.nse

## NSE scripts and timing templates (updated)

### Timing Templates
Nmap has timing templates `-T0` to `-T5` which affect scan speed and stealth:

- `-T0` (Paranoid) — Very slow, useful for IDS evasion.
- `-T1` (Sneaky) — Slow, less likely to trigger alerts.
- `-T2` (Polite) — Slower scan, reduces load on target.
- `-T3` (Default) — Balance between speed and accuracy.
- `-T4` (Aggressive) — Faster, may trigger IDS.
- `-T5` (Insane) — Maximum speed, very noisy, use only in lab/test.

**Example usage:**
```bash
nmap -T1 -p 80 --script http-title,http-headers target.com   # Slow, stealthy
nmap -T2 -p 80 --script http-title,http-headers target.com   # Polite, slightly faster
```

### Notes
- `-T1` or `-T2` are often used during testing of live targets where stealth is preferred.
- Higher timing templates (`-T4`, `-T5`) are for fast scans in controlled environments or labs.
- Combining timing with NSE scripts can help balance speed vs detection risk:
```bash
nmap -T2 -sV --script "http-headers,http-enum" -p 80 target.com
```

## Advanced techniques: vulnerability scans, evasion, spoofing & integrations

### Vulnerability scans (NSE `vuln` scripts)
**Example command:**
```bash
nmap -sV -p 80 --script "vuln" -oA scan_vuln target.com
```
- `-sV` — service/version detection (needed to give context for vuln checks).
- `-p 80` — target specific port (adjust as needed).
- `--script "vuln"` — runs the vulnerability-checking category of NSE scripts (many scripts in this category test for known CVEs or misconfigurations).
- `-oA scan_vuln` — save output in all formats for reporting/importing.

**Notes:**
- Some users write custom scripts (e.g. `--script vulnrs.nse`) — only run scripts you trust and understand. A wrong or malicious script can damage a target.
- Vulnerability scripts can be intrusive — **only run with explicit written authorization**.

---

### Firewall & IDS/IPS evasion (what nmap can do and limitations)
**Common Nmap techniques:**
- **Timing templates:** `-T0`..`-T5` — use `-T1` or `-T2` for stealthier scans to reduce IDS hits. (See timing section.)
- **Fragment packets:** `-f` — split probes into smaller fragments to evade simple packet filters. Limited effectiveness against modern IDS.
- **Decoys:** `-D RND:10` or `-D victim,decoy1,decoy2` — hide the real scanner among decoy sources. May confuse logs but can be illegal and noisy.
- **Source port trick:** `--source-port 53` — sets source port to a value that may be allowed by poorly-written firewalls (e.g. DNS). Effectiveness is situational.
- **Use of `-S` and `-e` for source IP/interface:** advanced raw packet options to set source IP or interface — usually limited and risky.

**Limitations & safety:**
- Evasion techniques often **fail** against modern, stateful firewalls, IDS/IPS systems, cloud load balancers and WAFs.
- They may generate **false confidence** and increase legal risk. Always include the customer/stakeholder in test scope and get written permission for evasive techniques.
- Some techniques can break connectivity or cause collateral impact on shared infrastructure.

**Example — stealthy + decoy (use only with permission):**
```bash
sudo nmap -sS -p 80 --script http-headers -T1 -f -D RND:5 --top-ports 100 -oA stealthy juice-shop.example.com
```
This runs a SYN scan targeting port 80, fragments packets, uses a random set of decoys, and uses a slow timing profile.

---

### MAC address spoofing (bypass simple MAC filters)
**Command:**
```bash
sudo nmap --spoof-mac 00:11:22:33:44:55 -sS -p 80 target
```
- `--spoof-mac` accepts explicit MAC addresses or vendor names (e.g., `--spoof-mac Cisco`) and only affects the local-link layer.
- **Important:** MAC spoofing only works on the same local network segment. It does **not** bypass filters farther upstream (across routers or over the internet).
- You may also need to bring your interface down/up or use `ip link set dev eth0 address <mac>` depending on your system.

**When:** Useful when testing local networks that enforce MAC-based allowlists. Only use on systems you control or have permission to test.

---

### Zombie (Idle) scan and IP spoofing
**Idle scan (zombie) — `-sI`**
```bash
nmap -sI <zombie-host> -p 80 target
```
- `-sI` performs an Idle scan using a third-party "zombie" host to proxy probes and conceal scanning origin. Nmap uses predictable IPID behavior from the zombie to infer port states on the target.
- **Requirements:** a suitable zombie with predictable IPID behavior and the ability for the scanner to observe or query it. Increasingly rare to find reliable zombies.
- **When to use:** very stealthy and can hide the origin of scans, but fragile and depends on network conditions.

**IP spoofing (`-S`)**
```bash
sudo nmap -S 1.2.3.4 -p 80 target
```
- `-S <addr>` sets the source IP of probes (spoof). This can be used to test certain filtering behaviour but **you usually won't get replies** back to your scanner unless you control the spoofed address or sniff on-path.
- IP spoofing can be disruptive and is likely illegal without explicit authorization. Modern networks and ISP filtering often drop spoofed traffic.

---

### Integrating Nmap with other tools
**1. Metasploit (import Nmap results)**
- Save Nmap output as XML: `nmap -sV -oX scan.xml target`
- In `msfconsole`:
```
msf6> db_import scan.xml
msf6> hosts
msf6> services
```
- Metasploit will import hosts and services from the XML output and can suggest/exploit relevant modules.

**2. SearchSploit / Exploit-DB**
- Take `-sV` service/version output and search locally with `searchsploit "nginx 1.14"` or format results for manual review.

**3. Nikto / Dirbuster / Gobuster**
- Use Nmap HTTP enumeration results (`http-enum`, `http-headers`, `http-title`) to prioritize paths. For deeper directory fuzzing:
```bash
gobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt -t 50
```

**4. Burp Suite**
- Use Nmap to discover hosts/paths and then load targets into Burp for interactive web testing.

**5. Automation workflow example (quick playbook):**
1. `nmap -sS -sV --top-ports 100 -oA scan_basic target`  
2. `msfconsole` -> `db_import scan_basic.xml`  
3. `searchsploit` on discovered versions  
4. Run `gobuster` or `nikto` against discovered web endpoints  
5. Use Burp for in-depth testing and proof-of-concept exploits

---

### Ethics, legality & safe practice (short)
- **Always** obtain explicit, written authorization before running intrusive scans, evasion, spoofing, or exploitation. Include scope, time window, allowed IP ranges, and contact points.
- Avoid testing targets hosted on shared cloud platforms (e.g., Heroku, AWS) unless you own the app or have clear authorization from the owner — you may impact other tenants.
- Maintain logs, save outputs (`-oA`), and include disclaimers in reports.

---
