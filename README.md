

# Casino SEO Poisoning: Complete Guide to Detection, Removal, and Prevention

> The growing threat of SEO theft and how to protect your server (Real-world case study on a VPS with AlmaLinux 9 and CWP)

---

## 📈 The Scale of the Problem

SEO theft for online casinos is growing at an alarming scale. We are no longer dealing with isolated attacks from "lone hackers"—we are talking about industrial-scale operations orchestrated by highly organized cybercrime groups with significant technical capabilities and financial backing.

These groups operate globally, using advanced techniques to redirect users—primarily from Thailand—to illegal gambling platforms. What you are about to read is based on a real-world infection scenario on a VPS server running AlmaLinux 9 with CWP (Control Web Panel).

---

## 1. What is Casino SEO Poisoning?

SEO Spam or SEO Poisoning is a cyberattack where criminals compromise legitimate websites to hijack their search engine authority and promote illegal products, such as unregulated online casinos (targeting audiences in Southeast Asia, notably Thailand) or counterfeit goods.


## How it Works in Practice

LEGITIMATE WEBSITE (Victim)  

         ↓
  [SERVER COMPROMISE]  

         ↓
MALICIOUS APACHE MODULE INSTALLED. 

         ↓
DETECTS GOOGLEBOT OR THAI IP ADDRESS. 

         ↓
INJECTS SPAM CONTENT (CLOAKING)  

         ↓
GOOGLE INDEXES FAKE PAGES. 

         ↓
TRAFFIC IS REDIRECTED. 

         ↓
ILLEGAL CASINO (PROFIT)  



## The Evolution of the Threat
SEO theft has evolved from simple spam injections into coordinated operations by organized groups leveraging:
Custom compilation tools
Native system-level backdoors (operating beneath the application layer, beyond standard PHP/WordPress files)
Monetization frameworks structured as "SEO fraud-as-a-service"

## 2. What Damage is Caused to Legitimate Website Owners?

# Impact on the Victim (Legitimate Website)
Impact
Description
Google Penalization
The domain receives algorithmic or manual actions, losing search positions.
Loss of Organic Traffic
Drastic drop in legitimate visitor metrics and search rankings.
Loss of Credibility
Visitors distrust the domain due to suspicious redirects or content.
Financial Losses
Immediate decline in conversions and organic revenue streams.
Reputational Harm
Brand association with illegal gambling activities.

# Advantage for the Attacker
Revenue Generation: Affiliate commissions from illegal Thai casino conversions.
Pay-per-Action: Payouts for every redirected user who registers or deposits.
Industrial Scale: Automated deployment attacking hundreds of server environments simultaneously.
## 3. How to Detect and Test if Your Site is Compromised

🔍 What Appears in Google Search?
This is usually the first visible sign of an attack:
Searching for site:yoursite.com on Google reveals heavily modified titles and meta descriptions promoting online casinos, sports betting, or Thai keywords such as "คาสิโน" (casino) or "พนัน" (gambling).
Google indexes non-existent URLs outside your original site structure (e.g., [yoursite.com/casino-online-thai/](https://yoursite.com/casino-online-thai/) or [yoursite.com/slot-gacor/](https://yoursite.com/slot-gacor/)).
Clicking these search results (especially via a Thai IP address or while simulating Googlebot) immediately redirects users to illegal gambling platforms.
Your Click-Through Rate (CTR) plummets because search snippets no longer match your brand's actual content.

⚠️ The Modern Attack Challenge (Real Case Study)
Important Note: In this specific real-world case, the attackers did not modify hosting accounts, plugins, themes, or individual PHP files. They targeted the underlying operating system directly by installing a custom-compiled malicious module into Apache itself.
The attacker avoided leaving plain-text strings in standard configuration files. Because the compromise occurred at the web server layer, they compiled a rogue Apache module with a dual purpose:
System Backdoor: Maintains persistent root access to the server.
SEO Spam Injector: Infiltrates malicious content exclusively when processing requests from Googlebot or Thai IP addresses.

Why is this so difficult to detect?

🔒 Compiled Binary Code: Standard grep searches for plain-text keywords like "casino" or "spam" fail.
🕵️ Selective Activation (Cloaking): Triggers only when the User-Agent is identified as Googlebot or the request originates from a target IP range.
📝 No Direct Access Logs: The injection happens dynamically in memory; no dedicated URLs exist in access logs.

🎭 Function Masking: Malicious routines execute within legitimate, standard Apache API function names.
Technical Detection Methods
A. Apache Integrity Verification (CWP / AlmaLinux)
Run package verification checks on the customized web server binaries:

# Essential command for CWP servers
rpm -V cwp-httpd


B. Debugging with mod_dumpio
To track dynamic payload injections, configure Apache's dumpio module to inspect input and output buffer sizes:
# Enable inside httpd.conf
LoadModule dumpio_module modules/mod_dumpio.so

DumpIOInput On
DumpIOOutput On


Compare input object lengths versus output object lengths to detect injected text payloads.
C. Hash Verification of Apache Modules
Real-World Case Study — Compromised Module on AlmaLinux 9:
Module Version
File Size
SHA-256 Checksum
Infected
44,346 bytes
75b41b3b9f60bede2d921da12db3cdc78027c774e51ec4619ec6271dd5f4e5ed
Official
36,296 bytes
f8766431dec0af4f10575a2a10b9b63b624961d062d5f4aa13b6a72bd573349f

Note: A comparative symbol analysis between the infected binary and the official version showed that both exported identical function symbols. The attacker did not declare new suspicious function names, but the discrepancy in size and checksum confirmed internal binary modifications.
D. JavaScript Payload Analysis
On HTML pages served by the compromised Apache instance, the following injected routine was isolated:
// Function named in Chinese Pinyin - strong indicator of threat group origin
function yinCangZhuZhanWaiLian() {
    // Translation: "Hide external links on the main site"
    // Malicious redirection logic executes here
}


E. Quick CLI Testing with cURL (Simulating Googlebot)
Because the malware checks request headers before injecting spam, you can force this condition via curl and compare HTTP responses directly.
Run these commands on your terminal:
1. Test using a standard User-Agent (regular visitor):
curl -s -I "https://yoursite.com/" | head -n 10


Expected Result: Returns HTTP/1.1 200 OK with standard page Content-Length.
2. Test simulating Googlebot (the execution trigger):
curl -s -I -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" "https://yoursite.com/" | head -n 10


Expected Result (if infected): Returns HTTP/1.1 301 / 302 (redirect) to an external domain, or a substantially larger Content-Length due to injected HTML/JavaScript.
3. Download and inspect response bodies side-by-side:
# Download page content as seen by Googlebot
curl -s -L -A "Googlebot" "https://yoursite.com/" > page_googlebot.html

# Download page content as seen by a normal user
curl -s -L "https://yoursite.com/" > page_normal.html

# Compare file sizes
ls -la page_*.html

# Search for casino strings or the Chinese JavaScript function name
grep -i "casino\|slot\|yinCangZhuZhanWaiLian\|apk.app" page_googlebot.html


If the second request reveals unexpected redirects or scripts absent from the clean request, the web server binary environment is compromised.  

## 4. How to Remove the Malware
🔄 Recommended Remediation Approach
CRITICAL: When malware operates at the compiled Apache module layer (typically in /usr/lib64/httpd/modules/), attempting to manual-patch the binary file is unsafe. The only reliable resolution is a full reinstallation of the web server package.
Step-by-Step Recovery on AlmaLinux 9 with CWP
Confirm module-level infection using the verification and hashing steps outlined above.
Completely reinstall the web server binaries:
# Reinstall standard CWP custom httpd package
yum reinstall cwp-httpd -y

# Alternatively, execute a complete package purge and reinstall:
yum remove cwp-httpd -y. 

yum install cwp-httpd -y


Manually verify /usr/lib64/httpd/modules/:
Check all .so binaries against known-good repository checksums.
In this incident, the infected binary was disguised as mod_auth_form.so.
Re-verify SHA-256 hashes after package installation to ensure files match package manager defaults.
Post-installation steps:
Restart Apache: systemctl restart httpd
Flush all edge-caching mechanisms (e.g., Cloudflare, Varnish, Nginx proxy layers).
Submit a re-indexing request via Google Search Console.
Server-wide post-cleanup audit:
Inspect all crontabs: crontab -l and /etc/cron.d/
Check persistent systemd units: systemctl list-unit-files
Audit temporary storage directories: /tmp and /var/tmp
5. Prevention and Hardening. 

## 🔐 Server-Level Security Controls
System Maintenance:
# Ensure operating system packages remain up-to-date
dnf update -y


Automated Web Server Verification:
# Automate weekly package integrity checks via cron
rpm -V cwp-httpd > /var/log/apache_integrity_check.log


Module Integrity Baseline:
Maintain an off-site manifest of official binary SHA-256 hashes.
Run daily comparative automated scripts to alert on hash drift.
Access Control Protocol:
Restrict SSH access strictly to Public-Key Authentication (disable password auth).
Enforce Two-Factor Authentication (2FA) across web management control panels (CWP).
Restrict administrative interfaces behind IP allowlists.
Change default operational ports (SSH, CWP control panel).  

## 🛡️ Application Security & Monitoring
ModSecurity Rules: Deploy rules blocking connection attempts to known malicious proxy destinations or C2 infrastructure.
File System Integrity Monitoring (FIM): Monitor changes to critical system binary folders using tools like AIDE or Tripwire.
Off-Site Backups: Maintain isolated, read-only remote server backups taken independently of the local hypervisor.
Log Auditing: Continuously inspect HTTP server logs (/var/log/httpd/) for unusual traffic spikes originating from regions outside your customer base.  

## 📋 Known Malicious Domains (Block via ModSecurity / Firewall)
Add these domains—extracted directly from injected payloads during this incident—to your perimeter blocklists or ModSecurity custom rulesets:  

bengkel69apk.app
ka789apk.app
tt789apk.dev
zk6apk.app
tkp188apk.app
gelay88apk.app
666l-a.com
sauditotapk.app
6rt-s.com
osg888-a.com
rp88-n.com
v89.io
1yk1.net/
33zkapk.app
ttt8882.com
PHL789-a.ph
sogoslot1.com
sl999link.com
ke7.io
salju4dapk.app
9n9napk.dev
P222-a.ph
yyrr-a.org
6666igames.com
nk666apk.app
gacormax-b.com
bb0303-a.com
PH1-a.ph
na77-s.com
api66-a.com
JKT88-n.com
janda4dapk.app
km777-w.com
PH54-a.ph
jkt88-b.com
v89-a.com
fw66apk.net
rk55apk.app
garuda55apk.app
koin138apk.app
juragan77apk.app
rajalangitapk.app
mega388apk.app
rpok-a.com
poros77apk.app
33ag-a.con
slot97apk.app
avatar808apk.app
cocol88apk.app
9yk1.com/
spn88-b.com

## Security Reminder: 
Immediately rotate all administrative secrets (root passwords, database credentials, FTP users, and panel tokens) after completing malware removal.
