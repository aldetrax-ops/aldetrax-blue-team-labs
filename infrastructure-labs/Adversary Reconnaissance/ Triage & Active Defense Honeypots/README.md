Triage & Active Defense Honeypots
This project documents the implementation of a localized security testing sandbox designed to analyze web application vulnerabilities and evaluate active defense mechanisms. By configuring an isolated network environment, this lab demonstrates how offensive exploitation techniques can be monitored, logged, and ultimately mitigated using defensive deception tools.

Environment & Infrastructure Design
To ensure testing safety, the entire infrastructure is built within a closed-loop virtual network, preventing any malicious traffic from escaping into production networks.

Hypervisor: VMware Workstation

Attacker & Server Target: Kali Linux VM (Configured with host-only networking via a custom VMnet1 adapter)

Access Layer: SecureCRT terminal emulator connected via SSH to the Kali Linux CLI for high-efficiency script management and log tracking.

Web Stack Initialization
To host the target applications without system port conflicts, the default system web server is disabled, and an isolated local development environment is started:
# Stop the default Apache service to free up Port 80
sudo systemctl stop apache2

# Start the XAMPP suite to launch the specialized Apache and MySQL instances
sudo /opt/lampp/lampp start
the target environment is accessible via a localized browser link pointing to the host's custom IP address hosting the OWASP Mutillidae II testbed.

Phase 1: Web Application Vulnerability Assessment
Two primary attack vectors from the OWASP Top 10 are executed against the local testbed to study how poorly written input processing behaves under exploitation.

1. Authentication Bypass via SQL Injection (SQLi)
The target application utilizes a raw SQL query string to validate user logins:
SELECT * FROM users WHERE username = 'input' AND password = 'input';
Because user input is directly concatenated into the database engine without sanitization, an attacker can input a specific logic payload into the username and password fields to manipulate the statement logic: ' OR '1'='1
classic payload used in SQL Injection (SQLi) attacks to manipulate backend databases
2. Local File Disclosure via XML External Entities (XXE)
Web applications that process raw XML documents without disabling external parsing can be tricked into reading host architecture files. By interacting with an online XML validator component on the testbed, the following malicious payload is submitted:
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE core [  
  <!ELEMENT core ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<core>&xxe;</core>
How it works:
The server's XML engine reads the custom data type definition (<!ENTITY xxe SYSTEM "file:///etc/passwd" >). When the parser processes the text tag &xxe;, it fetches the local system password file (/etc/passwd) from the root operating system and displays user details directly in the web browser output.

Phase 2: Active Deception & Honeypot Monitoring a defensive deception tool is deployed to catch automated scanning utilities like Nmap before an adversary finds legitimate entry points.
1. Deploying the Fake Service
Using Pentbox, an active honeypot is manually initialized with administrative privileges to listen on standard network ports.
# Navigate to the tool library
cd /home/kali/pentbox-1.8

# Elevate privileges to manage low-level network sockets
sudo su

# Execute the honeypot interface
./pentbox.rb

Triage & Intrusion Detection Capture

# Run a quick service discovery scan
nmap -v 192.168.101.X

# Attempt a direct raw connection to the open mail port
telnet 192.168.101.X 25
While the external scanning tool shows a generic black console waiting for input, the honeypot framework immediately identifies unauthorized behavior. SecureCRT captures real-time warning logs containing the attacker's source IP address, target port choice, and timestamp signatures.
