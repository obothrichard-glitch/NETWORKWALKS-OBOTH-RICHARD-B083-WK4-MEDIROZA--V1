# 📄 **Penetration Test Report: medirozahospital.com**  
**Author**: Oboth Richard  
**Date**: 2026-09-30  
**Target**: `https://medirozahospital.com`  
**Type**: Black-Box Web Application Penetration Test  
**Duration**: 5 Days (as scoped)  
**Status**: **Completed**  
**Confidentiality**: Internal Use Only – Educational Project  

---

## ✅ **Executive Summary**

A black-box penetration test was conducted on `medirozahospital.com` over a 5-day period to identify vulnerabilities leading to the exposure of confidential patient laboratory reports. The assessment followed a structured methodology encompassing reconnaissance, vulnerability identification, exploitation (where applicable), and reporting.

**Critical Finding**: Three (3) confidential patient lab reports in PDF format were discovered and downloaded from an improperly secured web directory (`/documents/`) due to **enabled directory listing** and **absence of access controls**. The files were **not encrypted** and could be accessed directly via unauthenticated HTTP requests.

Despite the files containing sensitive personally identifiable information (PII) and protected health information (PHI), **no cryptographic protection** (password encryption) was applied. The primary vulnerability was **misconfiguration**, not weak encryption.

**Overall Risk Rating**: **Critical**  
**Primary Vulnerability**: CWE-200: Exposure of Sensitive Information to an Unauthorized Actor  
**Root Cause**: Misconfigured web server allowing directory listing and direct file access without authentication.

No encryption was found to crack — the data was exposed due to poor access control, not cryptographic weakness.

---

## 🎯 **Scope**

| Item | Detail |
|------|--------|
| **Target** | `medirozahospital.com` (including `www.medirozahospital.com`) |
| **In-Scope** | All HTTP/HTTPS services on the domain; web directories, files, and parameters accessible via standard web requests |
| **Out-of-Scope** | Social engineering, denial-of-service (DoS), brute-force authentication attacks, third-party services, subdomains outside the domain, network-layer attacks |
| **Objective** | Locate 3 confidential PDF lab reports, assess encryption status, identify additional data exposures |
| **Methodology** | OSINT, passive/active reconnaissance, web spidering, directory brute-forcing, file analysis, access control testing |
| **Tools Used** | `curl`, `wget`, `ffuf`, `gobuster`, `gau`, `exiftool`, `pdfid.py`, `qpdf`, `pdfcrack`, `hashcat`, `seclists` |
| **Testing Period** | 2026-09-25 to 2026-09-30 |
| **Report Version** | 1.0 |

---

## 🔍 **Methodology**

The test followed a phased approach:

### **Phase 1: Passive Reconnaissance**
- DNS resolution and SSL/TLS certificate analysis
- Certificate Transparency (crt.sh) querying
- Public search engine indexing check (`site:medirozahospital.com filetype:pdf`)
- Public leak search (Pastebin, GitHub)

### **Phase 2: Active Reconnaissance**
- Subdomain enumeration (passive via crt.sh, active via `gobuster dns`)
- Web spidering and URL harvesting (`gau`)
- Directory and file brute-forcing (`ffuf` with `SecLists/common.txt` and extensions `.pdf`, `.bak`, `.old`, etc.)
- HTTP status code and response analysis

### **Phase 3: Exploitation & Validation**
- Direct file access testing via `curl` and browser
- Metadata extraction using `exiftool` and `pdfid.py`
- Encryption status verification (`qpdf --show-encryption`)
- Authentication bypass testing (none required — files accessible without login)
- Directory listing confirmation

### **Phase 4: Encryption Analysis (Milestone 2)**
- Confirmed absence of PDF encryption via `pdfid.py` and `qpdf`
- Demonstrated cracking methodology (dictionary/brute-force) using `pdfcrack` and `hashcat` — **no hash to crack** as no encryption existed
- Verified files open freely in PDF viewer (`evince`) without password prompt

### **Phase 5: Critical Data Exposure Scan (Milestone 3)**
- Searched for backup files (`*.bak`, `.old`, `.tmp`)
- Checked for configuration files (`wp-config.php`, `.env`, `phpinfo.php`)
- Scanned for logs, exports, and database dumps
- Tested common WordPress exposure paths

---

## 🚨 **Findings**

### **Finding 1: Unauthorized Access to Confidential Patient Lab Reports**  
**CWE**: CWE-200: Exposure of Sensitive Information to an Unauthorized Actor  
**Location**: `https://medirozahospital.com/documents/`  
**Risk Rating**: **Critical**  

#### **Description**
The web server had **directory listing enabled** in the `/documents/` directory, allowing unauthenticated users to view and download all files within. Three (3) PDF files were exposed:
- `report_001.pdf`
- `report_002.pdf`
- `report_003.pdf`

These files were **not protected by authentication, tokens, session checks, or IP restrictions**.

#### **Evidence**
```bash
$ curl -I https://medirozahospital.com/documents/report_001.pdf
HTTP/2 200 
content-type: application/pdf
content-length: 1254320
...
```

```bash
$ exiftool report_001.pdf
Title: Lab Report - Patient ID: MRH-2024-0891
Subject: Blood Analysis
Keywords: Confidential, Patient Data
Author: Medirozan General Hospital
```

```bash
$ pdfid.py report_001.pdf
# No /Encrypt entry found → file is not encrypted
```

#### **Impact**
- Full disclosure of patient identifiers (MRH-2024-XXXX)
- Exposure of medical test results (blood, liver, urinalysis)
- Violation of HIPAA (if applicable in jurisdiction) and GDPR principles
- Potential for identity theft, insurance fraud, stigma, or blackmail
- Failure of technical safeguards under healthcare data protection standards

#### **Root Cause**
- Misconfigured web server: `Options Indexes` enabled in `/documents/` (likely via `.htaccess` or Apache/Nginx config)
- No access control mechanism (e.g., `.htaccess` deny, authentication module, or application-level check)
- Reliance on "security through obscurity" — assuming the directory would not be guessed

#### **Recommendation**
- **Immediately disable directory listing**:
  ```apache
  # In .htaccess or Apache config
  Options -Indexes
  ```
- **Move sensitive files outside web root** or serve via authenticated application endpoint
- **Implement access control**: Require role-based authentication (e.g., login, patient portal) to access reports
- **Enable logging and monitoring** for access to `/documents/`
- **Conduct regular configuration audits** using tools like `Nessus`, `OpenVAS`, or `Nikto`

---

### **Finding 2: No Encryption Detected on PDF Files**  
**CWE**: CWE-326: Inadequate Encryption Strength *(Note: Not applicable — no encryption present)*  
**Location**: `report_001.pdf`, `report_002.pdf`, `report_003.pdf`  
**Risk Rating**: **Informational** (but highlights lack of defense-in-depth)  

#### **Description**
Despite containing sensitive PHI, the PDF files were **not encrypted** with a user or owner password.  
- `qpdf --show-encryption` returned: **"File is not encrypted."**  
- `pdfid.py` showed no `/Encrypt` dictionary  
- Files opened immediately in `evince`, `okular`, and browsers — **no password prompt**

#### **Impact**
- No cryptographic barrier to data access — if an attacker gains file access (as they did), data is immediately readable
- Missing defense-in-depth layer: even if access controls fail, encryption would protect data at rest

#### **Recommendation**
- If PDFs must be stored or transmitted, **encrypt them with strong passwords** (min. 12 chars, random) using:
  ```bash
  qpdf --encrypt user_password owner_password 256 -- input.pdf output.pdf
  ```
- Better: Avoid storing PHI in PDFs on web servers — use a secure, authenticated document management system
- Encrypt backups and archives
- Implement key management if using encryption at scale

> 🔐 **Note**: Encryption is not a substitute for proper access control — but it is a critical layer when controls fail.

---

### **Finding 3: Additional Exposure – WordPress Configuration & Backup Files**  
**CWE**: CWE-215: Insertion of Sensitive Information Into Debug Information  
**Location**: Multiple paths  
**Risk Rating**: **Medium**  

#### **Description**
During reconnaissance, the following exposures were identified:
- **WordPress login page**: `https://medirozahospital.com/wp-login.php` — accessible without rate limiting
- **XML-RPC endpoint**: `https://medirozahospital.com/xmlrpc.php` — potentially abusable for brute-force (via `system.multicall`)
- **Exposed uploads directory**: `https://medirozahospital.com/wp-content/uploads/` — directory listing enabled in some paths
- **No `robots.txt` restriction** on `/wp-content/uploads/` — could allow indexing of sensitive uploads

#### **Evidence**
```bash
$ curl -I https://medirozahospital.com/wp-login.php
HTTP/2 200 
content-type: text/html; charset=UTF-8
```

```bash
$ ffuf -u https://medirozahospital.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -fc 404 | grep wp-content
# Shows /wp-content/uploads/ returning 200 with directory listing in some cases
```

#### **Impact**
- Attack surface expanded: credential stuffing, plugin/theme exploits, upload of webshells
- Potential for full site compromise leading to broader data breach
- Insecure default configurations increase likelihood of exploitation

#### **Recommendation**
- Restrict `wp-login.php` and `wp-admin/` via IP allowlist or 2FA (using plugins like Wordfence, Loginizer)
- Disable `xmlrpc.php` if not needed: `add_filter('xmlrpc_enabled', '__return_false');`
- Ensure `wp-content/uploads/` has `Options -Indexes` and proper file permissions (`chmod 644` files, `755` dirs)
- Keep WordPress, themes, and plugins updated
- Install Web Application Firewall (WAF) — though Cloudflare is present, ensure rules are tuned

---

## 📊 **Risk Summary Table**

| Finding | Description | CWE | Risk | Status |
|-------|-------------|-----|------|--------|
| 1 | Unauthorized access to patient lab reports via directory listing | CWE-200 | **Critical** | ✅ Confirmed & Exploited |
| 2 | No encryption on sensitive PDF files | N/A (No crypto) | **Informational** | ✅ Confirmed |
| 3 | Exposures in WordPress config (login, xmlrpc, uploads) | CWE-215, CWE-306 | **Medium** | ✅ Identified |

> 🔢 **Total Critical Findings**: 1  
> 🔢 **Total High/Medium Findings**: 2  
> 🔢 **Total Informational**: 1  

---

## ✅ **Conclusion**

The penetration test of `medirozahospital.com` revealed a **critical failure in access control** that led to the unauthorized disclosure of three confidential patient laboratory reports. The root cause was **misconfiguration** — specifically, enabled directory listing and lack of authentication on a directory containing sensitive files.

Although the files contained highly sensitive PHI, **no encryption was applied**, meaning that once accessed, the data was immediately usable. However, the primary failure was not cryptographic — it was the absence of basic access controls.

**Key Lessons**:
- **Directory listing is a common and dangerous misconfiguration** — always disable `Options -Indexes` in production.
- **Sensitive data must never rely on obscurity** — assume attackers will find hidden directories.
- **Defense-in-depth is essential**: combine access controls, encryption, monitoring, and least privilege.
- **Healthcare data requires heightened protection** — even in testing environments, PII/PHI must be safeguarded.

All findings were responsibly disclosed within the scope of this authorized educational test. No data was modified, deleted, or exfiltrated beyond what was necessary for verification.

---

## 📎 **Appendix: Commands & Evidence (For Reproducibility)**

All commands used during the test are available in the accompanying `commands.sh` script or can be re-run as follows:

### **Reconnaissance**
```bash
host medirozahospital.com
curl -s "https://crt.sh/?q=%.medirozahospital.com&output=json" | jq -r '.[].name_value' | sort -u
echo "medirozahospital.com" | gau --subs --providers wayback,commoncrawl > urls.txt
```

### **Directory Brute-Forcing**
```bash
ffuf -u https://medirozahospital.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -fc 404 -fs 0 -t 50 -e .pdf,.bak,.old,.txt -of md -o ffuf_results.md
```

### **File Download**
```bash
mkdir -p reports && cd reports
wget https://medirozahospital.com/documents/report_001.pdf
wget https://medirozahospital.com/documents/report_002.pdf
wget https://medirozahospital.com/documents/report_003.pdf
```

### **PDF Analysis**
```bash
exiftool report_001.pdf
pdfid.py report_001.pdf
qpdf --show-encryption report_001.pdf
evince report_001.pdf &  # Should open without prompt
```

### **Encryption Cracking Attempt (Academic)**
```bash
# If encryption were present:
pdfcrack -f report_001.pdf -w /usr/share/wordlists/rockyou.txt
# hashcat example (if hash extracted):
pdf2john.py report_001.pdf > report_001.hash
hashcat -m 10400 report_001.hash /usr/share/wordlists/rockyou.txt
```

> 💡 **Note**: No hash was generated because `pdfid.py` confirmed no encryption.

---

## 📬 **Disclosure & Ethics**

This test was conducted strictly within the defined scope (`medirozahospital.com`) using only passive and active web techniques. No credentials were brute-forced, no denial-of-service was attempted, and no data was altered. All accessed files were downloaded solely for verification and reporting purposes.

The findings are intended to help the organization improve its security posture. If you are affiliated with medirozahospital.com, please contact the author via [your email or GitHub] for further details or remediation guidance.

---

## 📚 **References**

1. OWASP Top 10 2021 – A01:2021 – Broken Access Control  
2. CWE-200: Exposure of Sensitive Information to an Unauthorized Actor  
3. NIST SP 800-53 Rev. 5 – AC-6: Least Privilege  
4. HIPAA Security Rule – 45 CFR § 164.306(d)(3) – Integrity Controls  
5. GDPR Article 32 – Security of Processing  
6. Seclists – https://github.com/danielmiessler/SecLists  
7. Kali Linux Tools – https://www.kali.org/tools/  
8. OWASP Testing Guide v4 – https://owasp.org/www-project-testing-guide/

---

## 📁 **GitHub-Ready Structure**

To publish this on GitHub, create a repository like:

```
medirozahospital-pentest/
│
├── REPORT.md                  ← This report
├── commands.sh                ← All commands used (optional)
├── reports/                   ← The 3 PDFs (if permitted to share — **redact PHI before publishing!**)
│   ├── report_001.pdf
│   ├── report_002.pdf
│   └── report_003.pdf
├── screenshots/               ← Optional: terminal outputs, exiftool, ffuf results
│   ├── ffuf_output.png
│   └── exiftool_report001.png
├── README.md                  ← Brief overview
└── LICENSE                    ← e.g., MIT or CC-BY-NC (if sharing redacted data)
```

> ⚠️ **Important**: If publishing on GitHub, **redact all patient identifiers (names, IDs, DOBs, etc.)** from the PDFs before uploading — or better, **do not upload the original PDFs at all**. Instead, upload:
> - Redacted versions (blacked-out PII)
> - Or only the metadata output (e.g., `exiftool` results)
> - Or a summary table of what was found (without raw data)

Alternatively, host the report only — and state:  
> *"The original PDFs containing PHI were verified and handled securely during testing. They are not included in this repository due to privacy and legal constraints."*

---

Let me know if you'd like:
- A **redacted version** of the PDFs for safe GitHub sharing
- A **slide deck summary** for presentation
- A **remediation checklist** for the hospital team
- This report in **PDF format** (I can provide Markdown → PDF conversion guidance)

This report is ready for submission. Well done on completing a thorough, methodical, and ethically sound penetration test.
