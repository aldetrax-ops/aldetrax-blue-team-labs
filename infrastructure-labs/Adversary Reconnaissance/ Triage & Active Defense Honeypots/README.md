Markdown
## Triage & Active Defense Honeypots

This repository documents the technical implementation of a local security testing sandbox used to evaluate web application vulnerabilities and deploy active network deception mechanisms. The core focus of this project is to analyze how common exploit vectors operate from an offensive standpoint and demonstrate how defensive tools can be used to log and triage unauthorized network reconnaissance.

---

### Environment Architecture

To ensure safety, the entire lab environment is isolated within a closed-loop virtual network, preventing test traffic from interacting with production systems.

* **Virtualization Host:** VMware Workstation
* **Operating System:** Kali Linux VM running on a Host-Only network adapter (Custom VMnet1)
* **Access Configuration:** Remote command-line administration managed through SecureCRT for efficient terminal execution and live log tracking.

**Web Service Setup**
Before launching the target testbeds, the default host web service is stopped to prevent port assignment conflicts, allowing the training application environment to boot cleanly:

```bash
sudo systemctl stop apache2
sudo /opt/lampp/lampp start
Phase 1: Web Application Vulnerability Assessment
Black-box penetration testing was conducted against OWASP Mutillidae II web testbeds to identify and exploit high-risk input processing flaws.

1. Authentication Bypass via SQL Injection (SQLi)
The application's login form was evaluated for input validation flaws where user input is directly concatenated into database queries.

Target Query Logic: SELECT * FROM users WHERE username = 'input' AND password = 'input';

Exploitation Payload: ' OR '1'='1

Mechanism: Submitting this payload forces the database logic to evaluate an empty string condition alongside a constant true statement (1=1). Because the query logic resolves to true globally, the application bypasses the authentication controls and exposes user records.

2. System Configuration Disclosure via XML External Entities (XXE)
Application parameters that parse raw XML documents were tested to determine if external entity resolution was securely disabled.

Exploitation Payload:

XML
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE core [  
  <!ELEMENT core ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<core>&xxe;</core>
Mechanism: The malicious XML payload forces the target web server parser into processing a custom system entity reference. When parsed, the backend server executes the local file request and returns the contents of the restricted /etc/passwd system configuration file directly to the browser view.

Phase 2: Active Deception & Honeypot Monitoring
To implement proactive defensive measures, a decoy infrastructure was deployed to detect automated scanning and unauthorized access attempts before they reach critical assets.

1. Honeypot Deployment
Using the Pentbox framework under administrative privileges, a simulated network listener was bound to a standard communication port:

Bash
cd /home/kali/pentbox-1.8
sudo su
./pentbox.rb
The utility was configured to host a simulated Simple Mail Transfer Protocol (SMTP) service listening directly on Port 25.

2. Triage & Reconnaissance Capture
An external machine on the same isolated subnet initiated a service discovery scan and a raw connection attempt to simulate standard attacker behavior:

Bash
nmap -v 192.168.101.X
telnet 192.168.101.X 25
While the connecting terminal remains blank, the defensive framework instantly logs the interaction. SecureCRT actively captures and parses the real-time adversary reconnaissance signatures, providing the source IP address, connection timestamp, and target port metadata.

Engineering Remediation Standards
Defending production environments against these tactical vectors requires enforcing strict secure development guidelines:

SQLi Mitigation: Implement strictly bound parameters using Prepared Statements or Object-Relational Mapping (ORM) frameworks to decouple user data input from executable query command structures.

XXE Mitigation: Explicitly configure all system XML parser libraries to completely disable external Document Type Definitions (DTDs) and external entity resolution.
