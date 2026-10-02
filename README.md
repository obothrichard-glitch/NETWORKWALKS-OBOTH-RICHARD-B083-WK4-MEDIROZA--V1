# MEDIROZA GENERAL HOSPITAL

## WEB APPLICATION PENETRATION TESTING REPORT

**Assessment Type:** Black-Box Web Application Penetration Test

**Target:** `https://medirozahospital.com`

**Assessment Scope:** Mediroza General Hospital public-facing web application

**Testing Environment:** Authorized Training/Laboratory Environment

**Testing Tools:** Burp Suite Community Edition, Firefox Developer Tools, Networkwalks Hash Calculator, Networkwalks Password Cracker, ExifTool, pdfinfo, qpdf, Gobuster, FFUF and cURL

**Prepared By:** Oboth Richard

**Date:** 30th October 2026

---

# 1. Executive Summary

A black-box penetration test was conducted against the Mediroza General Hospital web application to identify security weaknesses affecting authentication, confidential patient documents and supporting web infrastructure.

The assessment followed an attack-chain approach, beginning with reconnaissance and identification of publicly exposed application functionality, followed by authentication testing, controlled exploitation, document analysis and investigation of information discovered during the assessment.

The assessment identified several security weaknesses that could be chained together to obtain unauthorized access to sensitive information.

The major findings were:

1. Weak authentication controls on the patient portal.
2. Unauthorized access to confidential patient laboratory reports after authentication compromise.
3. Weak passwords protecting encrypted PDF laboratory reports.
4. Sensitive information disclosed through PDF metadata.
5. An exposed legacy database backup accessible through a publicly browsable directory.

The most significant exposure was the publicly accessible legacy database backup. The backup contained sensitive staff and shareholder information, including personally identifiable information, salary information and ownership information.

The assessment demonstrates that security weaknesses do not necessarily have to be individually catastrophic to create significant risk. In this case, weaknesses in authentication, document protection, metadata handling and legacy-file management formed a chain that increased the overall impact.

---

# 2. Scope

## 2.1 Target

The assessment was limited to:

`https://medirozahospital.com`

Testing was restricted to the target domain and directly related subordinate paths within the authorized scope.

## 2.2 Out-of-Scope Activities

The following activities were not performed:

* Social engineering
* Denial-of-service testing
* Destructive attacks
* Testing unrelated external systems
* Attacks against third-party infrastructure outside the authorized target

Testing was conducted only within the permitted assessment scope.

---

# 3. Objectives

The primary objectives of the assessment were to:

* Identify publicly exposed application entry points.
* Assess the security of the patient portal authentication mechanism.
* Determine whether authentication controls could be bypassed or compromised.
* Assess authorization controls protecting patient laboratory reports.
* Analyze the protection applied to downloaded PDF documents.
* Identify sensitive information disclosed through document metadata.
* Investigate exposed legacy files and directories.
* Determine the potential impact of discovered vulnerabilities.
* Provide practical remediation recommendations.

---

# 4. Methodology

The assessment followed a black-box penetration-testing methodology consisting of:

1. Reconnaissance
2. Attack-surface discovery
3. Authentication testing
4. Manual request analysis
5. Controlled credential testing
6. Restricted-area verification
7. Confidential-document analysis
8. PDF encryption assessment
9. Metadata analysis
10. Investigation of discovered infrastructure references
11. Impact assessment
12. Remediation recommendations

Burp Suite was used to intercept and analyze web requests and responses. Burp's HTTP History was used to review application traffic and compare response status codes, lengths and other characteristics.

Burp Repeater was used where manual modification and re-submission of individual requests was required.

---

# 5. Tools Used

| Tool                          | Purpose                                                                             |
| ----------------------------- | ----------------------------------------------------------------------------------- |
| Burp Suite Community Edition  | HTTP interception, request analysis, Repeater and controlled authentication testing |
| Firefox Developer Tools       | Browser-side reconnaissance and request analysis                                    |
| Networkwalks Hash Calculator  | Extraction of PDF encryption hashes                                                 |
| Networkwalks Password Cracker | Dictionary-based recovery of PDF encryption passwords                               |
| ExifTool                      | PDF metadata analysis                                                               |
| pdfinfo                       | PDF information and metadata inspection                                             |
| qpdf                          | PDF decryption after authorized password recovery                                   |
| Gobuster                      | Directory and resource discovery                                                    |
| FFUF                          | Web-content discovery                                                               |
| cURL                          | Manual HTTP requests and response verification                                      |

The Networkwalks Academy password-recovery workflow specifically uses its Hash Calculator to obtain the PDF hash and then the Password Cracker to recover the password using a dictionary attack.

---

# 6. Findings Summary

| ID | Finding                                                           | Severity |
| -- | ----------------------------------------------------------------- | -------- |
| F1 | Weak / Brute-Forceable Patient Portal Authentication              | High     |
| F2 | Unauthorized Access to Confidential Patient Laboratory Reports    | High     |
| F3 | Weak Passwords Protecting Encrypted PDF Reports                   | Medium   |
| F4 | Sensitive Metadata Disclosure in Patient PDFs                     | Medium   |
| F5 | Unauthenticated Exposure of Legacy HR/Shareholder Database Backup | Critical |

---

# 7. Finding F1 — Weak Authentication

**Severity:** High

## Description

The patient portal login functionality was identified at:

`/patient/login.php`

Initial testing included examination of the login parameters for possible injection-related weaknesses. SQL injection testing did not produce a successful result.

The authentication mechanism was subsequently tested using controlled credential testing in Burp Suite.

The application did not appear to implement effective account lockout, rate limiting or CAPTCHA protections. This allowed repeated authentication attempts without an effective automated defence.

## Evidence

The response to an unsuccessful authentication attempt returned an HTTP 200 response containing the normal incorrect-login response.

A successful authentication attempt produced a different response pattern, including an HTTP 302 redirect toward the authenticated portal.

This difference provided a reliable method of distinguishing unsuccessful and successful authentication attempts.

## Impact

An attacker able to perform repeated authentication attempts could potentially discover a valid credential and obtain access to functionality intended for authenticated users.

## Recommendation

* Implement rate limiting.
* Implement account lockout or progressive authentication delays.
* Implement CAPTCHA or equivalent bot mitigation where appropriate.
* Enforce strong password requirements.
* Implement multi-factor authentication for accounts accessing patient information.
* Monitor and alert on repeated authentication failures.
* Avoid relying only on HTTP status codes as an authentication-security mechanism.

---

# 8. Finding F2 — Unauthorized Access to Confidential Patient Laboratory Reports

**Severity:** High

## Description

Following successful authentication, the patient portal exposed a section containing confidential laboratory reports.

The authenticated area contained three downloadable PDF laboratory reports.

The documents contained sensitive patient-related information and were protected by PDF passwords.

## Impact

Compromise of the portal authentication mechanism resulted in access to confidential patient documents.

This demonstrates the direct relationship between the authentication weakness identified in F1 and the confidentiality of patient information.

## Recommendation

* Enforce strict server-side authorization for every patient document.
* Ensure users can access only documents belonging to their authorized account.
* Verify authorization on every document-download request.
* Do not rely solely on the secrecy of document URLs.
* Log access to sensitive patient documents.
* Monitor unusual document-download activity.
* Consider stronger authentication for users with access to sensitive records.

---

# 9. Finding F3 — Weak Passwords Protecting PDF Reports

**Severity:** Medium

## Description

The three downloaded laboratory reports were protected using PDF encryption.

The assessment did not treat the presence of encryption as proof that the documents were adequately protected. Instead, the strength of the passwords protecting the encryption was assessed.

The following workflow was used:

1. The encrypted PDF was obtained from the authorized test environment.
2. The PDF was submitted to the **Networkwalks Hash Calculator**.
3. The tool generated the PDF encryption hash in `$pdf$` format.
4. The extracted hash was submitted to the **Networkwalks Password Cracker**.
5. A dictionary-based password recovery attempt was performed.
6. The recovered password was used to decrypt the authorized test document.
7. The decrypted document was subsequently examined for metadata and other information.

The Networkwalks Academy documentation describes this same workflow for its educational PDF-password recovery lab.

## Result

All three test PDF passwords were recoverable using the available dictionary-based approach, indicating that the passwords had insufficient entropy.

Recovered passwords are intentionally **not included in this report**.

## Impact

PDF encryption provided an additional security layer, but the protection was weakened substantially by the use of predictable passwords.

An attacker who obtained the encrypted PDFs could potentially recover their passwords using freely available password-recovery techniques.

## Recommendation

* Generate long, random passwords.
* Avoid names, dates, common words and predictable patterns.
* Use unique credentials for individual documents.
* Consider authenticated document-delivery mechanisms instead of relying exclusively on PDF passwords.
* Protect the original documents at the application/server level before they are downloaded.
* Review password-generation procedures used by the document-generation system.

---

# 10. Finding F4 — Sensitive PDF Metadata Disclosure

**Severity:** Medium

## Description

After authorized recovery of the PDF contents, metadata analysis was performed using ExifTool and pdfinfo.

The documents contained information associated with the internal document-generation environment.

The analysis identified:

* Internal CMS information.
* An internal username.
* An operational comment referencing a legacy `/old/` location.

The metadata therefore disclosed information that was not intended to be part of the patient-facing document.

## Security Impact

Metadata can unintentionally disclose:

* Internal usernames.
* Software names and versions.
* Internal operational comments.
* File paths.
* Information about migration or deployment processes.
* Details that may help an attacker map internal infrastructure.

In this assessment, the metadata disclosure provided an important lead for further investigation.

## Recommendation

Before releasing documents to external users:

* Remove unnecessary PDF metadata.
* Remove Author fields containing internal usernames.
* Remove Comments and internal operational notes.
* Review Producer and Creator fields.
* Sanitize generated documents automatically.
* Include document metadata review in the secure-development process.

---

# 11. Finding F5 — Publicly Accessible Legacy Database Backup

**Severity:** Critical

## Description

Investigation of the `/old/` path revealed that directory listing was enabled.

A legacy database backup was accessible without authentication.

The exposed file was:

`mediroza_db_backup_2019.sql`

The backup contained sensitive organizational information.

The identified data included:

### Staff information

The database contained staff records including fields such as:

* Full name
* Job title
* Department
* Email address
* Telephone number
* National identification information
* Monthly salary
* Date joined

### Shareholder information

The database also contained shareholder information including:

* Shareholder name
* Percentage ownership
* Number of shares
* Share class

## Impact

This finding represents a severe confidentiality exposure because the database backup was accessible without authentication.

The exposed information could potentially facilitate:

* Identity-related attacks.
* Targeted phishing.
* Privacy violations.
* Employee targeting.
* Financial information disclosure.
* Corporate intelligence gathering.
* Further attacks against the organization.

## Recommendation

Immediate actions should include:

1. Remove the exposed backup from the public web root.
2. Ensure backups cannot be accessed directly through HTTP.
3. Disable directory listing.
4. Review the entire web root for additional legacy files.
5. Rotate credentials that may have appeared in the backup.
6. Assess whether exposed personal information requires formal incident handling or notification.
7. Establish a secure backup-storage process.
8. Implement a lifecycle policy for old backups.
9. Regularly scan production systems for forgotten files and directories.

---

# 12. Attack Chain

The assessment demonstrated how several weaknesses could be chained together:

```text
Public Web Application
        │
        ▼
Patient Portal Login
        │
        ▼
Weak Authentication Controls
        │
        ▼
Credential Compromise
        │
        ▼
Authenticated Patient Portal
        │
        ▼
Confidential PDF Reports
        │
        ▼
PDF Password Recovery
        │
        ▼
PDF Metadata Analysis
        │
        ▼
Internal Path /old/
        │
        ▼
Public Directory Listing
        │
        ▼
Legacy Database Backup
        │
        ▼
Sensitive HR + Shareholder Information
```

The significance of the assessment is therefore not limited to individual vulnerabilities. The findings formed a chain in which information obtained at one stage enabled further investigation at the next stage.

---

# 13. Risk Assessment

| Finding | Severity | Main Risk                                                       |
| ------- | -------- | --------------------------------------------------------------- |
| F1      | High     | Account compromise through weak authentication controls         |
| F2      | High     | Unauthorized access to patient laboratory information           |
| F3      | Medium   | Recovery of protected PDF contents                              |
| F4      | Medium   | Disclosure of internal technical information                    |
| F5      | Critical | Public exposure of HR, PII, payroll and shareholder information |

---

# 14. Overall Security Assessment

The assessment identified multiple weaknesses across authentication, document security, information disclosure and web-server configuration.

The most significant issue was the exposure of the legacy database backup through the production web server.

The assessment also demonstrated that confidential documents should not be considered adequately protected merely because they are password-protected. The effectiveness of the protection depends heavily on password strength and the security of the surrounding application.

The combination of authentication weaknesses, sensitive document exposure, weak document passwords, metadata leakage and an exposed legacy backup created a substantially larger security risk than any one issue considered independently.

---

# 15. Remediation Plan

## Priority 1 — Remove exposed data

* Remove `/old/` from the public web root.
* Remove all database backups from publicly accessible directories.
* Search the server for other `.sql`, `.bak`, `.zip`, `.old`, `.backup` and temporary files.
* Disable directory listing.

## Priority 2 — Secure authentication

* Implement rate limiting.
* Implement account lockout/progressive delays.
* Add MFA for sensitive accounts.
* Strengthen password requirements.
* Monitor authentication failures.

## Priority 3 — Protect patient documents

* Enforce authorization server-side.
* Verify authorization for every download.
* Prevent predictable document URLs.
* Log access to sensitive documents.

## Priority 4 — Improve PDF security

* Use cryptographically strong random passwords.
* Use unique passwords for individual documents.
* Avoid predictable patient information as passwords.
* Consider authenticated download links.

## Priority 5 — Sanitize metadata

* Remove internal usernames.
* Remove comments.
* Remove internal paths.
* Review PDF creator/producer information.
* Automate metadata sanitization before external distribution.

## Priority 6 — Improve backup management

* Store backups outside the web root.
* Restrict backup access.
* Encrypt sensitive backups.
* Maintain an inventory of production backups.
* Establish backup retention and deletion procedures.
* Perform regular external attack-surface reviews.

---

# 16. Conclusion

The penetration test demonstrated that the Mediroza web application contained several security weaknesses affecting authentication, confidentiality and information disclosure.

The most serious exposure resulted from a legacy database backup being accessible through the public web server without authentication.

The assessment also demonstrated the importance of examining the complete security chain rather than testing individual controls in isolation. A weak authentication mechanism led to access to confidential documents, weak PDF passwords reduced the effectiveness of document encryption, and metadata contained information that provided a lead to an exposed legacy directory.

The recommended remediation actions should prioritize removal of publicly accessible sensitive data, strengthening authentication, enforcing server-side authorization, improving document protection, sanitizing metadata and implementing secure backup-management practices.

All testing described in this report was performed within the authorized assessment scope and was intended for security assessment and educational purposes.
