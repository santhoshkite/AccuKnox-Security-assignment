# Security Assessment Report

## 1. Summary

This report goes into detail about the vulnerabilities found at the endpoint `http://www.itsecgames.com`. The most critical issue identified is an outdated and vulnerable version of OpenSSH (6.7p1) exposed to the internet on port 22. This poses a significant risk, as it is susceptible to multiple known exploits, including a username enumeration flaw (`CVE-2018-15473`). A medium-severity finding revealed that the website operates over unencrypted HTTP, exposing user data to interception and man-in-the-middle attacks. Additionally, several low-severity vulnerabilities were discovered, including a missing anti-clickjacking header, information disclosure through ETag headers, and the exposure of default server files, all of which could aid an attacker in further reconnaissance.

## 2. Scope and Methodology

### 2.1 Scope
The assessment was limited to a black-box scan and analysis of the publicly accessible web application at the following endpoint:
*   `http://www.itsecgames.com`

### 2.2 Methodology
The assessment was performed using non-intrusive, publicly available tools to conduct infrastructure and web application reconnaissance. The process followed a standard methodology:
1.  Network and Service Discovery
2.  Web Server Vulnerability Scanning
3.  Analysis and Reporting

**Tools Used:**
*   Nmap
*   Nikto

---

## 3. Findings and Recommendations

### 3.1 Finding 1: Open TCP port 22 with known vulnerabilities
**Severity:** High

**Description:** 
A network scan reported an open SSH port on the public internet running an outdated and vulnerable version OpenSSH 6.7p1. This version of OpenSSH has multiple CVEs such as `CVE-2018-15473`, `CVE-2015-5600`. The `CVE-2018-15473` is a critical one; this vulnerability allows a remote attacker to determine if a specific username is valid on the server without needing to guess a password. By sending a specially malformed request, the server will behave differently depending on whether the username exists or not.

**Evidence:**
<img width="972" height="266" alt="Screenshot 2025-09-21 at 10 27 52 AM" src="https://github.com/user-attachments/assets/fb55ed00-c4ee-4682-9703-9a9e6f2c8a99" />
> We can clearly see the version of OpenSSH running on the domain is 6.7p1.

**Recommendation:**
*   **Upgrade Software:** The most effective control is to upgrade the OpenSSH server package to the latest stable version. This will patch all known vulnerabilities.
*   **Use Key-Based Authentication:** Disable password-based authentication and use SSH key-based authentication. This mitigates the risk of brute-force and password spraying attacks, rendering the username enumeration flaw far less useful.

---

### 3.2 Finding 2: Communication is not secure
**Severity:** Medium

**Description:** 
The web application at `http://www.itsecgames.com` is insecure because it uses HTTP instead of HTTPS, leaving all transmitted data unencrypted. This vulnerability makes the application and its users susceptible to various attacks, including Man-in-the-Middle (MitM) attacks, eavesdropping, session hijacking, and data tampering, as sensitive information like login credentials and personal data can be easily intercepted and compromised. Implementing HTTPS with TLS/SSL encryption is crucial to secure communication, ensuring data confidentiality, integrity, and authenticity.

**Evidence:**

<img width="472" height="266" alt="Screenshot 2025-09-21 at 10 29 17 AM" src="https://github.com/user-attachments/assets/eee079b5-a58e-4188-8170-02cb7c5f2f10" />

> The site is served over `http://`.

**Recommendation:**
*   **Implement HTTPS:** Secure the web application by obtaining and configuring an SSL/TLS certificate and redirecting all HTTP traffic to HTTPS.

---

### 3.3 Finding 3: Missing Anti-clickjacking Header
**Severity:** Low

**Description:** 
The response does not protect against clickjacking attacks. It should include either a Content-Security-Policy header with the frame-ancestors directive or an X-Frame-Options header.

**Evidence:**

<img width="472" height="266" alt="Screenshot 2025-09-21 at 10 29 17 AM" src="https://github.com/user-attachments/assets/eee079b5-a58e-4188-8170-02cb7c5f2f10" />

> We can see there are no `X-Frame-Options` or `Content-Security-Policy` with `frame-ancestors 'self'` in the HTTP headers to prevent clickjacking.

**Recommendation:**
*   **Implement Anti-Clickjacking Headers:** Configure the web server to include either the `X-Frame-Options` header (set to `DENY` or `SAMEORIGIN`) or a `Content-Security-Policy` header with the `frame-ancestors` directive (e.g., `frame-ancestors 'self'`). This prevents the site from being embedded in iframes on other domains, mitigating clickjacking attacks.

---

### 3.4 Finding 4: File Path Disclosure via ETag Header
**Severity:** Low

**Description:** 
The web server’s ETag header for the root path (/) includes sensitive information, specifically the inode number (e43) and file size (5d7959bd3c800). This information leakage, as described in CVE-2003-1418, can be exploited by attackers to gain insights into the server’s file system structure. While it does not directly compromise data or server control, it could aid in further reconnaissance and targeted attacks.

**Evidence:**
<img width="1437" height="266" alt="Screenshot 2025-09-21 at 10 31 46 AM" src="https://github.com/user-attachments/assets/6e226d16-f5a5-45ee-a865-a8e8df7c5bc1" />
> ETag header reveals inode, file size, and mtime.

**Recommendation:**
*   **Reconfigure ETag Headers:** Configure the web server to remove inode information from ETag headers. This can typically be achieved by modifying the server's configuration (e.g., `FileETag MTime Size` in Apache) to use a weaker ETag validation mechanism or by disabling ETag generation if not strictly required.

---

### 3.5 Finding 5: Sensitive Files Found
**Severity:** Low

**Description:** 
The Apache default file `README` was found accessible via the `/icons/README` path. While this specific file might not contain highly sensitive information on its own, its presence indicates that default configuration files, which can sometimes disclose server versions, directory structures, or other potentially useful information for an attacker, are publicly exposed. This can aid in further reconnaissance and potentially lead to more targeted attacks.

**Evidence:**
<img width="1437" height="266" alt="Screenshot 2025-09-21 at 10 32 10 AM" src="https://github.com/user-attachments/assets/839b994a-6719-4876-bdbc-d3949675dedd" />
> The file is accessible at `/icons/README`.

**Recommendation:**
*   **Remove Default Files:** Remove or restrict access to default server files and directories that are not intended for public access. This typically involves configuring the web server to prevent directory listing and to deny access to specific files or directories that could contain sensitive information.

---

## 4. Conclusion

The security assessment of `http://www.itsecgames.com` revealed several vulnerabilities that collectively weaken its overall security posture. The most immediate threat is the public-facing, vulnerable OpenSSH service, which requires immediate attention to prevent potential unauthorized access or information leakage. Furthermore, the lack of HTTPS encryption represents a significant risk to data confidentiality and integrity for all users of the site. The low-severity findings, while less critical individually, indicate a need for better server hardening and configuration management to reduce the application's attack surface. It is strongly recommended that the organization prioritize patching the OpenSSH server and implementing HTTPS. Subsequently, the remaining low-severity configuration issues should be addressed to ensure a comprehensive security posture and protect the application and its users from known threats.
