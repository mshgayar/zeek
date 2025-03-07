### Common Types of Zeek Logs and Associated Threats
```Zeek (formerly Bro) generates a variety of log files that provide deep visibility into network activity. ```
```These logs help in detecting different types of threats, anomalies, and malicious behaviors.```
```Below is a list of common Zeek log types and the security threats associated with each.```
![alt text](https://github.com/mshgayar/zeek/blob/main/Zeek%20logs%20table.png)

## 1. conn.log (Connection Log)
**Description:** 

* Records metadata about all observed network connections.
* Includes fields such as source/destination IPs, ports, protocol, duration, bytes sent/received.
* Threats Detected:
	* ✔️ Port Scanning & Reconnaissance: Multiple short-lived connections to different ports.
	* ✔️ DDoS Attacks: High volume of connections from a single or multiple sources.
	* ✔️ C2Communication: Unusual long-lived connections to unknown remote IPs.
* ✔️ Lateral Movement: Suspicious internal IP-to-IP connections.

🔹 Example Detection:

* A single host attempting to connect to 1000+ ports within a short period → Potential Port Scan.
* A high number of short-duration connections to different destinations → Possible DoS Attack.

--
## 2. dns.log (DNS Query Log)
**Description:**

* Logs all DNS queries and responses.
* Includes source/destination IP, query type, and resolved domain.
* Threats Detected:
	* ✔️ DNS Tunneling: Large DNS queries used for data exfiltration.
	* ✔️ C2 Domains: Queries to known malicious or newly registered domains.
	* ✔️ Fast Flux Attacks: Rapidly changing IP resolutions for the same domain.
	* ✔️ Phishing Attacks: Queries to typosquatted or suspicious domains.

🔹 Example Detection:

* High volume of queries to *.xyz or newly registered domains → Possible Malware C2.
* Base64-like encoded subdomains in DNS queries → Potential DNS Tunneling.


## 3. http.log (HTTP Log)
**Description:**

* Logs HTTP request metadata (host, user-agent, URI, response codes, referrer).
* Does not capture actual HTTP content.
* Threats Detected:
	* ✔️ Malicious URL Access: Requests to phishing or malware hosting websites.
	* ✔️ Exfiltration via HTTP: Large uploads from sensitive sources.
	* ✔️ Command Injection: Suspicious query parameters (e.g., cmd.exe, wget).
	* ✔️ User-Agent Anomalies: Non-standard or obfuscated user-agents (e.g., Python scripts, malware loaders).

🔹 Example Detection:

* Repeated access to hxxp://malicious-site.com/shell.php → Potential Web Shell Access.
* User-agent Mozilla/5.0 (Windows NT 6.1; Win64; x64) used by a Linux system → User-Agent Spoofing.

--

## 4. ssl.log (TLS/SSL Log)
**Description:**

* Logs SSL/TLS handshake information.
* Captures server name, certificate details, encryption method.
* Threats Detected:
	* ✔️ Self-Signed Certificates: Attackers using rogue certificates.
	* ✔️ Expired/Tampered Certificates: Potential MITM (Man-in-the-Middle) attacks.
	* ✔️ Weak Ciphers: Legacy or vulnerable TLS versions (e.g., TLS 1.0, SSLv3).
	* ✔️ Encrypted C2 Traffic: Suspicious TLS connections to rare/unclassified domains.

🔹 Example Detection:

* A host connecting to example[.]xyz using TLS 1.0 → Weak TLS Vulnerability.
* Repeated SSL connections with self-signed certificates → Possible C2 Encrypted Traffic.

--
## 5. files.log (File Transfer Log)
**Description:**

* Logs metadata about files transferred over various protocols (HTTP, FTP, SMB, SMTP).
* Includes file hash, MIME type, and extraction status.
* Threats Detected:
	* ✔️ Malware Delivery: Suspicious file downloads (.exe, .dll, .vbs, .js).
	* ✔️ Data Exfiltration: Large outbound file transfers.
	* ✔️ Rogue Executables: Unexpected executable transfers via email/web.

🔹 Example Detection:

* File download from hxxp://example.com/malware.exe → Possible Malware Infection.
* Outbound .zip files > 1GB to unknown IP → Possible Data Exfiltration.

--
## 6. smtp.log (Email Log)
**Description:**

* Captures metadata about SMTP (email) traffic.
* Logs sender, receiver, subject, and attachments.
* Threats Detected:
	* ✔️ Phishing Emails: Emails from fake domains with suspicious subjects.
	* ✔️ Malware Attachments: Executable or macro-enabled document attachments.
	* ✔️ Business Email Compromise (BEC): Emails impersonating executives.

🔹 Example Detection:

* Email from ceo@fake-company.com with a subject Urgent Invoice → Potential Phishing Attack.
* Attachment .docm with macros enabled → Possible Malware Attack.

--
## 7. weird.log (Anomalies & Protocol Violations)
**Description:**

* Logs traffic anomalies that do not conform to expected protocol behavior.
* Captures protocol violations, dropped packets, and unusual activities.
* Threats Detected:
	* ✔️ Evasion Techniques: Attackers manipulating protocol behavior.
	* ✔️ Malformed Packets: Potential reconnaissance or exploitation attempts.
	* ✔️ Excessive Packet Drops: Possible DoS attack.

🔹 Example Detection:

* bad_TCP_checksum on multiple packets from the same source → Possible IDS Evasion Attempt.
* unusual_large_dns_response → Potential DNS Spoofing or Tunneling.

--
## 8. notice.log (Security Alerts & Warnings)
**Description:**

* Stores alerts generated by Zeek's detection scripts.
* Includes predefined security policies or custom rules.
* Threats Detected:
	* ✔️ Rogue DNS Queries: Alert when a host queries a known bad domain.
	* ✔️ Suspicious File Hashes: Matches with VirusTotal or YARA rules.
