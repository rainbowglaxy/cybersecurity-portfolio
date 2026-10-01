# Web Application Penetration Testing Report

## Article Management System

**Author:** Weixing Liu  
**Institution:** Zhengzhou Normal University  
**Team:** Cybersecurity Training Group 1  
**Assessment completed:** September 24, 2026  
**Original report version:** 1.0  
**Edition:** Redacted English public edition, published with permission

This report documents a manual black-box assessment of an article management system in a course-authorized training environment. Five security issues were confirmed in the original assessment: three rated high, one medium, and one low. The overall risk condition was assessed as Level 3, “Severe,” under the course report's classification scheme.

The assessment was limited to the authorized lab instance. Temporary test content was removed after testing. No data deletion outside that cleanup, bulk automated scanning, or persistence activity was reported.

## Confidentiality Statement

The original report was classified as Commercial Confidential. It was prepared by Zhengzhou Normal University's cybersecurity training group for an authorized course lab. Its methods, findings, and screenshots were intended for teaching and security research. The source report prohibits copying, distributing, or commercially using its contents without the commissioning party's written permission and assigns responsibility for misuse to the user.

This public edition is shared with the publication permission confirmed by the author. It omits student identifiers, the live lab hostname and port, session values, and original images. Figure numbers are retained as English descriptions of the original evidence. The placeholder `https://lab.example.invalid` represents the assessed lab and is not a live target.

## Document Information

- **Project and document name:** Article Management System Penetration Testing Report.
- **Original document and project identifiers:** Withheld because they contain a student identifier.
- **Version:** 1.0.
- **Formal deliverable in the original report:** Yes.
- **Creation and revision date:** September 24, 2026.
- **Original confidentiality classification:** Commercial Confidential.
- **Approval required in the original report:** Yes.
- **Prepared by:** Weixing Liu.
- **Contact information:** Not provided in the source report.

### Revision History

| Version | Scope | Date | Author |
| --- | --- | --- | --- |
| 1.0 | Entire original report | September 24, 2026 | Weixing Liu |

## Executive Summary

The target was a typical PHP and MariaDB dynamic website running Apache/2.4.25 (Debian) and PHP/5.6.40. Testing was conducted manually through an internet-facing access point, designated JB in the course template. Browser observations and comparisons of HTTP requests were used to validate the findings.

Five security issues were confirmed: administrator login SQL injection causing authentication bypass; SQL injection in public article parameters; stored cross-site scripting in article publication; reflected cross-site scripting in search; and disclosure of server software versions. The original report rated the first three high, the search issue medium, and version disclosure low.

A separate check of 95 common username/password combinations did not find a valid weak credential. All attempts failed. This is a result for the tested dictionary, not proof that no weak credentials exist.

The system's risk condition during the assessment was classified as Severe. Database query construction, authentication logic, and input/output handling should receive priority remediation. Appendix A defines the course classification scheme.

**Table 0-1 — Assessment Findings**

| No. | Finding identifier | Summary | Original severity |
| --- | --- | --- | --- |
| 1 | 2026_20260924_sqli_01 | SQL injection at administrator login allows an administrator session without the correct password | High |
| 2 | 2026_20260924_sqli_02 | The public `show.php` and `list.php` `id` parameters allow SQL error disclosure and Boolean-based injection | High |
| 3 | 2026_20260924_xss_01 | Stored XSS in article content executes on the public page and can access cookies readable by JavaScript | High |
| 4 | 2026_20260924_xss_02 | The search `keywords` parameter is reflected into HTML without appropriate encoding | Medium |
| 5 | 2026_20260924_info_01 | HTTP response headers expose detailed Apache and PHP versions | Low |

## 1 Project Information

### 1.1 Commissioning Organization

**Organization:** Zhengzhou Normal University, cybersecurity training lab.  
**Contact:** Weixing Liu; student training project. Contact details were omitted in the source.

### 1.2 Assessment Team

**Team:** Cybersecurity Training Group 1.  
**Assessor:** Weixing Liu. The student identifier is withheld in this public edition.

## 2 Project Overview

### 2.1 Purpose

The purpose was to understand the security condition of the course lab's article management system. Within an authorized and controlled scope, I examined potential web vulnerabilities from an external user's perspective, validated their exploitability and impact, and proposed practical remediation while completing the course learning objectives.

### 2.2 Scope

The scope was limited to the running article management system lab instance, including its public pages and `/admin` interface. Networks, hosts, and data outside the authorized instance were excluded from scanning and attack.

| Application | Target representation in the public edition |
| --- | --- |
| Article management system, including public pages and `/admin` | `https://lab.example.invalid/` |

**Figure 2-1 — English evidence description.** The original screenshot shows the target application's home page in its normal state before testing.

### 2.3 Topology and Access Point

The lab was a cloud-hosted container instance reached directly over the internet from campus or public networks. No additional internal network topology was involved.

The template defines JA as internal corporate network access and JB as internet access. This assessment used JB. Findings were reachable through the internet-facing access point; the template's access-point convention did not call for reducing their assigned ratings.

## 3 Assessment Method

### 3.1 Tools

The assessment used manual browser validation in Microsoft Edge or an embedded browser, HTTP request comparisons using `curl`, source-code and response-header inspection, and screenshots. Common SQL injection and XSS validation techniques informed the work. No automated vulnerability scanner was used for bulk attacks.

### 3.2 Accounts and Weak Credential Checks

No legitimate administrator credentials were available at the start. Five common usernames—`admin`, `root`, `test`, `manage`, and `system`—were paired with 19 common passwords, producing 95 checks. Every attempt returned an incorrect-credentials message.

An administrator session was subsequently obtained through the SQL injection bypass described in Section 4.1.

### 3.3 System State Before and After Testing

Testing was controlled and non-destructive. The temporary article created to validate stored XSS, article ID 36, was deleted at the end. The public article count returned to the original four. The source report records no persistent changes to lab files, configuration, or data beyond the temporary test content and its cleanup.

## 4 Vulnerability Findings

Input points on public and administrator pages were tested manually, simulating an external internet user. The focus was SQL injection, authentication bypass, XSS, and information disclosure. The following sections preserve the original finding identifiers and evidence figure numbers.

### 4.1 Administrator Login SQL Injection and Authentication Bypass

**Severity:** High.  
**Identifier:** `2026_20260924_sqli_01`.  
**Endpoint:** `POST /admin/login.action.php`.  
**Access point:** JB, internet access.

#### Description

The administrator login processed the username without adequate validation or parameterized SQL. A crafted username changed the authentication query's logic and allowed access without knowing the correct password.

#### Details and Recorded Result

The username was set to `admin 'OR' 1` and an arbitrary incorrect password was submitted. The application redirected to `/admin/index.php`, displayed the injected username in its welcome message, and issued administrator-related cookies including `username`, `userid=1`, and `PHPSESSID`. Actual session values are omitted.

The assessor subsequently accessed category management, article management, announcements, messages, file management, and administrator account pages, confirming access to the administrative functions described in the source report.

#### Test Procedure and Evidence

1. Open `/admin/`, enter `admin 'OR' 1` as the username, and enter an arbitrary password.
2. Submit the login form and observe entry to the administrator home page.
3. Visit additional administrative functions to verify the resulting privilege level.

**Figure 4-1 — English evidence description.** The administrator login form contains the crafted username `admin 'OR' 1`.

**Figure 4-2 — English evidence description.** After submission, the application displays the administrator home page and a welcome message containing the injected username.

#### Remediation

1. Use prepared statements and bound parameters for login queries. Do not concatenate usernames into SQL.
2. Verify both the account identity and the password hash without permitting a crafted true condition to replace authentication.
3. Apply failed-login rate limits and record anomalous attempts.
4. Regression-test valid credentials, an incorrect password, and the crafted username after remediation.

### 4.2 SQL Injection in Public Article Parameters

**Severity:** High.  
**Identifier:** `2026_20260924_sqli_02`.  
**Endpoints:** `GET /show.php?id=` and `GET /list.php?id=`.  
**Access point:** JB, internet access.

#### Description

The `id` values on the article detail and category list pages were incorporated into database queries without adequate type validation or parameterization. SQL errors were reflected to the user, and Boolean conditions produced distinguishable page responses.

#### Details and Recorded Result

Appending a single quote to `show.php?id=33` produced a MariaDB syntax error containing `You have an error in your SQL syntax` and a reference to the quote near line 1. Appending a quote to `list.php?id=22` produced an error referring to `order by a.id desc` near line 7, exposing part of the query structure.

For a Boolean comparison, `id=33 AND 1=1` displayed the article normally with an approximately 5,263-byte response. With `id=33 AND 1=2`, the article disappeared and the response was approximately 2,474 bytes. The difference supports Boolean-based SQL injection. The report does not claim that database records were extracted through this technique.

#### Test Procedure and Evidence

1. Request `show.php?id=33'` and observe the database error.
2. Compare `id=33 AND 1=1` against `id=33 AND 1=2` and observe the presence or absence of article content.
3. Request `list.php?id=22'` and observe the second injection point's error.

**Figure 4-3 — English evidence description.** A single quote in the `show.php` parameter produces a MariaDB error.

**Figure 4-4 — English evidence description.** The article remains visible when the appended condition `AND 1=1` is true.

**Figure 4-5 — English evidence description.** The article content disappears when `AND 1=2` is false.

**Figure 4-6 — English evidence description.** The `list.php` error exposes an `ORDER BY` fragment from the query.

#### Remediation

1. Validate identifier parameters as integers and reject invalid values.
2. Use parameterized queries consistently across database access.
3. Disable detailed SQL error output in production; keep diagnostic details in protected logs.
4. Grant the application's database account only the privileges it needs.

### 4.3 Stored XSS in Article Publication

**Severity:** High, as assigned in the original report.  
**Identifier:** `2026_20260924_xss_01`.  
**Input endpoint:** `POST /admin/article.action.php`, field `content`.  
**Execution location:** Public `/show.php` article page.  
**Access point:** JB. Insertion occurred through the administrator interface; subsequent public visitors could encounter the stored content.

#### Description

The administrator's article editor used FCKeditor. The `content` field did not adequately sanitize an `img` element with an `onerror` event handler. Submitted script-bearing content was stored in the database and executed when a visitor opened the article.

#### Details and Recorded Result

The application performed simple checks for terms such as `alert` and removed backslashes. An attempted encoded form such as `\u0061lert` therefore lost its backslash and did not operate as intended. A payload using an `img` error handler without that keyword bypassed the described check:

```html
<img src=x onerror="document.body.innerHTML='<h1>STORED XSS EXECUTED</h1><p>Cookie: '+document.cookie+'</p>'">
```

Opening the article rewrote the page and displayed cookies accessible through `document.cookie`, including the cookie names `username`, `userid`, and `PHPSESSID`. Actual cookie values are withheld. This demonstrated script execution and local cookie readability; the report does not document transmission of cookies to an external recipient.

#### Test Procedure and Evidence

1. Open the administrator's article publication page.
2. Submit the test content through the `content` field; a temporary article titled “XSS Verification Test” appears in the article list.
3. Inspect the public page source and confirm that the script-bearing markup was stored.
4. Open the article and observe script execution and cookie display.

**Figure 4-7 — English evidence description.** The administrator article publication page includes the FCKeditor rich-text editor.

**Figure 4-8 — English evidence description.** The temporary XSS verification article appears in the article list after submission.

**Figure 4-9 — English evidence description.** The public page source contains the stored `img onerror` payload; backslash removal is noted in the original evidence.

**Figure 4-10 — English evidence description.** The public article displays “STORED XSS EXECUTED” and JavaScript-readable cookie data. Session values are omitted from this edition.

#### Remediation

1. Sanitize rich HTML with an allowlist-based sanitizer such as DOMPurify used in an appropriate, maintained deployment. Remove unapproved event handlers and script-capable URL schemes.
2. Encode ordinary user text according to its output context. For intentionally permitted rich HTML, preserve only sanitized, approved markup rather than relying on keyword filtering.
3. Set appropriate `HttpOnly`, `Secure`, and `SameSite` attributes on session cookies. These controls reduce certain impacts but do not fix XSS.
4. Add a suitable Content Security Policy as a defense-in-depth measure.

### 4.4 Reflected XSS in Public Search

**Severity:** Medium.  
**Identifier:** `2026_20260924_xss_02`.  
**Endpoint:** `GET /search.php?keywords=`.  
**Access point:** JB, internet access.

#### Description

The search page placed the supplied `keywords` value into HTML without appropriate encoding. A crafted search link could cause script-bearing markup to be interpreted in a visitor's browser.

#### Details and Recorded Result

When `keywords=<svg/onload=alert(1)>` was submitted, the source contained the unencoded value inside a font element:

```html
<font style="color:#F00"><svg/onload=alert(1)></font>
```

The unsafe reflection was confirmed by source inspection. The source report describes how the browser can interpret the markup and trigger its event; the listed figure evidence specifically records the reflection in the page source.

#### Test Procedure and Evidence

1. Perform a normal search and observe the displayed search term.
2. Submit `<svg/onload=alert(1)>` and inspect the source to confirm that the tag was not encoded.

**Figure 4-11 — English evidence description.** A normal search term is reflected in the results page.

**Figure 4-12 — English evidence description.** The search input appears unencoded in the HTML source, demonstrating the unsafe reflection.

#### Remediation

1. Apply context-appropriate output encoding to user input reflected into HTML.
2. Do not rely solely on blacklist filtering.
3. Deploy a suitable Content Security Policy as an additional layer of defense.

### 4.5 Server Version Disclosure

**Severity:** Low.  
**Identifier:** `2026_20260924_info_01`.  
**Location:** Home page HTTP response headers.  
**Access point:** JB, internet access.

#### Description

The server disclosed detailed software version strings in HTTP response headers. Such information can help an attacker identify relevant known issues.

#### Details and Evidence

A normal request to the home page returned:

```http
Server: Apache/2.4.25 (Debian)
X-Powered-By: PHP/5.6.40
```

The source report notes that these components were old. No further exploitation of their known vulnerabilities was performed in this assessment.

**Figure 4-13 — English evidence description.** Response headers disclose Apache/2.4.25 and PHP/5.6.40.

#### Remediation

1. Minimize the Apache server banner with suitable settings such as `ServerTokens Prod` and `ServerSignature Off`.
2. Disable PHP's `expose_php` setting.
3. Upgrade old components according to a maintenance plan. Hiding versions does not replace patching.

## 5 Overall Security Condition

**Overall classification:** Level 3, Severe.

The assessment identified several high-rated application defects: SQL injection that bypassed administrator login, error-based and Boolean-based injection in public parameters, and stored XSS that executed on the public page and accessed JavaScript-readable cookies.

Simple filtering of selected XSS keywords and backslashes was insufficient. Database parameterization and output handling were inadequate at the tested input points. The 95 weak-credential attempts were unsuccessful, while response headers disclosed old Apache and PHP versions.

The source report therefore assessed the application's condition as high risk or Severe during the testing period. Remediation should prioritize the recommendations in Sections 4.1–4.5. This classification is the course template's qualitative assessment, not a newly calculated CVSS score.

## 6 Course Project Reflection

Through this exercise, I completed the process from information gathering and vulnerability validation to evidence collection. I gained a practical understanding of SQL injection, authentication bypass, and stored and reflected XSS.

The login test showed how concatenating usernames into SQL can undermine authentication. The article parameter tests showed error disclosure and Boolean response differences caused by unsafe query construction. The XSS tests showed how unsafe output handling can turn input into executable browser content.

The exercise reinforced the importance of parameterized queries and context-aware output handling, and helped me understand that web security spans the frontend, backend, database, and operational configuration. I intend to continue studying cybersecurity and gaining experience through authorized labs and projects to prepare for internships and employment.

## Appendix A Security Risk Condition Levels

| Level | Condition | Meaning |
| --- | --- | --- |
| 1 | Good | The system operates normally, with no identified issues or only isolated low-risk issues. Maintain existing controls. |
| 2 | Warning | Vulnerabilities or security weaknesses exist. Apply targeted improvements to relevant network, host, application, and management controls. |
| 3 | Severe | Serious vulnerabilities or issues may substantially threaten normal operation. Prompt patching and strengthened protections are required. |
| 4 | Emergency | The system faces a serious security situation that may substantially harm organizational interests. Coordinate emergency defensive action with relevant security teams. |

The assessed application was assigned Level 3, Severe, in the original report.
