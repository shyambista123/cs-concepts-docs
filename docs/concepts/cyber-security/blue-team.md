# Blue Team - Defensive Security

Blue Team refers to the defensive side of cybersecurity. Blue team professionals focus on protecting systems, detecting threats, and responding to security incidents. They work to strengthen defenses and maintain the security posture of an organization.

## Core Responsibilities

### 1. Security Monitoring
Continuously monitor systems for suspicious activities and potential threats.

### 2. Incident Response
Respond to and mitigate security incidents when they occur.

### 3. Threat Intelligence
Gather and analyze information about potential threats.

### 4. Security Hardening
Strengthen systems against attacks through configuration and patching.

### 5. Vulnerability Management
Identify and remediate security weaknesses.

## Key Activities

### Security Information and Event Management (SIEM)

**Example: Setting up Log Monitoring**

```python
import re
from datetime import datetime

class LogMonitor:
    def __init__(self):
        self.failed_login_threshold = 5
        self.failed_attempts = {}
    
    def analyze_log(self, log_entry):
        # Parse log entry
        pattern = r'Failed login attempt for user (\w+) from IP (\d+\.\d+\.\d+\.\d+)'
        match = re.search(pattern, log_entry)
        
        if match:
            username = match.group(1)
            ip_address = match.group(2)
            
            # Track failed attempts
            key = f"{username}:{ip_address}"
            self.failed_attempts[key] = self.failed_attempts.get(key, 0) + 1
            
            # Alert if threshold exceeded
            if self.failed_attempts[key] >= self.failed_login_threshold:
                self.trigger_alert(username, ip_address)
    
    def trigger_alert(self, username, ip_address):
        alert = {
            'timestamp': datetime.now(),
            'severity': 'HIGH',
            'type': 'Brute Force Attack',
            'username': username,
            'ip_address': ip_address,
            'action': 'Block IP and notify admin'
        }
        print(f"ALERT: {alert}")
        self.block_ip(ip_address)
    
    def block_ip(self, ip_address):
        # Add to firewall blocklist
        print(f"Blocking IP: {ip_address}")

# Usage
monitor = LogMonitor()
logs = [
    "Failed login attempt for admin from IP 192.168.1.100",
    "Failed login attempt for admin from IP 192.168.1.100",
    "Failed login attempt for admin from IP 192.168.1.100",
    "Failed login attempt for admin from IP 192.168.1.100",
    "Failed login attempt for admin from IP 192.168.1.100",
]

for log in logs:
    monitor.analyze_log(log)
```

### Intrusion Detection System (IDS)

**Example: Simple Network Traffic Analyzer**

```python
from scapy.all import sniff, IP, TCP
from collections import defaultdict
import time

class SimpleIDS:
    def __init__(self):
        self.connection_count = defaultdict(int)
        self.port_scan_threshold = 10
        self.time_window = 60  # seconds
        self.scan_attempts = defaultdict(list)
    
    def detect_port_scan(self, packet):
        if packet.haslayer(TCP) and packet.haslayer(IP):
            src_ip = packet[IP].src
            dst_port = packet[TCP].dport
            current_time = time.time()
            
            # Track port access attempts
            self.scan_attempts[src_ip].append({
                'port': dst_port,
                'time': current_time
            })
            
            # Clean old entries
            self.scan_attempts[src_ip] = [
                attempt for attempt in self.scan_attempts[src_ip]
                if current_time - attempt['time'] < self.time_window
            ]
            
            # Check for port scan
            unique_ports = len(set(a['port'] for a in self.scan_attempts[src_ip]))
            
            if unique_ports >= self.port_scan_threshold:
                self.alert_port_scan(src_ip, unique_ports)
    
    def alert_port_scan(self, src_ip, port_count):
        print(f"⚠️  PORT SCAN DETECTED!")
        print(f"   Source IP: {src_ip}")
        print(f"   Ports scanned: {port_count}")
        print(f"   Action: Blocking IP and notifying security team")
    
    def analyze_packet(self, packet):
        self.detect_port_scan(packet)

# Usage (requires root/admin privileges)
# ids = SimpleIDS()
# sniff(prn=ids.analyze_packet, filter="tcp", count=100)
```

### Vulnerability Scanning

**Example: Basic Web Application Scanner**

```python
import requests
from urllib.parse import urljoin

class VulnerabilityScanner:
    def __init__(self, target_url):
        self.target_url = target_url
        self.vulnerabilities = []
    
    def check_sql_injection(self, url):
        """Test for SQL injection vulnerabilities"""
        payloads = ["'", "1' OR '1'='1", "'; DROP TABLE users--"]
        
        for payload in payloads:
            test_url = f"{url}?id={payload}"
            try:
                response = requests.get(test_url, timeout=5)
                
                # Check for SQL error messages
                sql_errors = [
                    "SQL syntax",
                    "mysql_fetch",
                    "ORA-",
                    "PostgreSQL",
                    "SQLite"
                ]
                
                for error in sql_errors:
                    if error.lower() in response.text.lower():
                        self.vulnerabilities.append({
                            'type': 'SQL Injection',
                            'severity': 'CRITICAL',
                            'url': test_url,
                            'payload': payload
                        })
                        return True
            except Exception as e:
                pass
        return False
    
    def check_xss(self, url):
        """Test for XSS vulnerabilities"""
        payloads = [
            "<script>alert('XSS')</script>",
            "<img src=x onerror=alert('XSS')>",
            "javascript:alert('XSS')"
        ]
        
        for payload in payloads:
            test_url = f"{url}?search={payload}"
            try:
                response = requests.get(test_url, timeout=5)
                
                if payload in response.text:
                    self.vulnerabilities.append({
                        'type': 'Cross-Site Scripting (XSS)',
                        'severity': 'HIGH',
                        'url': test_url,
                        'payload': payload
                    })
                    return True
            except Exception as e:
                pass
        return False
    
    def check_security_headers(self):
        """Check for missing security headers"""
        try:
            response = requests.get(self.target_url, timeout=5)
            headers = response.headers
            
            required_headers = {
                'X-Frame-Options': 'MEDIUM',
                'X-Content-Type-Options': 'MEDIUM',
                'Strict-Transport-Security': 'HIGH',
                'Content-Security-Policy': 'HIGH',
                'X-XSS-Protection': 'MEDIUM'
            }
            
            for header, severity in required_headers.items():
                if header not in headers:
                    self.vulnerabilities.append({
                        'type': 'Missing Security Header',
                        'severity': severity,
                        'header': header,
                        'recommendation': f'Add {header} header'
                    })
        except Exception as e:
            pass
    
    def scan(self):
        """Run all vulnerability checks"""
        print(f"Scanning {self.target_url}...")
        
        self.check_sql_injection(self.target_url)
        self.check_xss(self.target_url)
        self.check_security_headers()
        
        return self.vulnerabilities
    
    def generate_report(self):
        """Generate vulnerability report"""
        print("\n" + "="*60)
        print("VULNERABILITY SCAN REPORT")
        print("="*60)
        print(f"Target: {self.target_url}")
        print(f"Total Vulnerabilities Found: {len(self.vulnerabilities)}\n")
        
        for i, vuln in enumerate(self.vulnerabilities, 1):
            print(f"{i}. {vuln['type']} - Severity: {vuln['severity']}")
            for key, value in vuln.items():
                if key not in ['type', 'severity']:
                    print(f"   {key}: {value}")
            print()

# Usage
# scanner = VulnerabilityScanner("http://example.com")
# scanner.scan()
# scanner.generate_report()
```

### Security Hardening

**Example: System Hardening Checklist Script**

```python
import os
import subprocess

class SystemHardening:
    def __init__(self):
        self.checks = []
    
    def check_firewall_status(self):
        """Check if firewall is enabled"""
        try:
            result = subprocess.run(['ufw', 'status'], 
                                  capture_output=True, text=True)
            if 'active' in result.stdout.lower():
                self.checks.append(('Firewall', 'PASS', 'Firewall is active'))
            else:
                self.checks.append(('Firewall', 'FAIL', 'Firewall is not active'))
        except Exception as e:
            self.checks.append(('Firewall', 'ERROR', str(e)))
    
    def check_ssh_config(self):
        """Check SSH configuration"""
        ssh_config = '/etc/ssh/sshd_config'
        issues = []
        
        try:
            with open(ssh_config, 'r') as f:
                config = f.read()
                
                # Check for root login
                if 'PermitRootLogin yes' in config:
                    issues.append('Root login is enabled')
                
                # Check for password authentication
                if 'PasswordAuthentication yes' in config:
                    issues.append('Password authentication is enabled')
                
                # Check for empty passwords
                if 'PermitEmptyPasswords yes' in config:
                    issues.append('Empty passwords are permitted')
            
            if issues:
                self.checks.append(('SSH Config', 'FAIL', ', '.join(issues)))
            else:
                self.checks.append(('SSH Config', 'PASS', 'SSH is properly configured'))
        except Exception as e:
            self.checks.append(('SSH Config', 'ERROR', str(e)))
    
    def check_updates(self):
        """Check for available system updates"""
        try:
            result = subprocess.run(['apt', 'list', '--upgradable'], 
                                  capture_output=True, text=True)
            updates = len(result.stdout.split('\n')) - 2
            
            if updates > 0:
                self.checks.append(('System Updates', 'WARN', 
                                  f'{updates} updates available'))
            else:
                self.checks.append(('System Updates', 'PASS', 
                                  'System is up to date'))
        except Exception as e:
            self.checks.append(('System Updates', 'ERROR', str(e)))
    
    def check_password_policy(self):
        """Check password policy settings"""
        try:
            with open('/etc/login.defs', 'r') as f:
                config = f.read()
                
                # Check password age
                if 'PASS_MAX_DAYS\t99999' in config:
                    self.checks.append(('Password Policy', 'FAIL', 
                                      'Password expiration not set'))
                else:
                    self.checks.append(('Password Policy', 'PASS', 
                                      'Password policy configured'))
        except Exception as e:
            self.checks.append(('Password Policy', 'ERROR', str(e)))
    
    def run_audit(self):
        """Run all hardening checks"""
        print("Running Security Hardening Audit...\n")
        
        self.check_firewall_status()
        self.check_ssh_config()
        self.check_updates()
        self.check_password_policy()
        
        self.generate_report()
    
    def generate_report(self):
        """Generate hardening report"""
        print("="*70)
        print("SYSTEM HARDENING AUDIT REPORT")
        print("="*70)
        
        passed = sum(1 for c in self.checks if c[1] == 'PASS')
        failed = sum(1 for c in self.checks if c[1] == 'FAIL')
        warnings = sum(1 for c in self.checks if c[1] == 'WARN')
        
        print(f"\nSummary: {passed} Passed | {failed} Failed | {warnings} Warnings\n")
        
        for check, status, message in self.checks:
            status_symbol = {
                'PASS': '✓',
                'FAIL': '✗',
                'WARN': '⚠',
                'ERROR': '!'
            }
            print(f"{status_symbol.get(status, '?')} {check:20} [{status:5}] {message}")

# Usage (requires root privileges)
# auditor = SystemHardening()
# auditor.run_audit()
```

## Blue Team Tools

### 1. SIEM Solutions
- **Splunk**: Enterprise log management and analysis
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **IBM QRadar**: Security intelligence platform
- **ArcSight**: Security event management

### 2. Intrusion Detection/Prevention
- **Snort**: Open-source IDS/IPS
- **Suricata**: Network threat detection
- **OSSEC**: Host-based IDS
- **Zeek (Bro)**: Network security monitor

### 3. Endpoint Protection
- **CrowdStrike**: Endpoint detection and response
- **Carbon Black**: Endpoint security platform
- **Microsoft Defender**: Built-in Windows protection
- **Sophos**: Comprehensive endpoint security

### 4. Network Monitoring
- **Wireshark**: Packet analyzer
- **Nagios**: Infrastructure monitoring
- **Zabbix**: Enterprise monitoring
- **PRTG**: Network monitoring tool

### 5. Vulnerability Management
- **Nessus**: Vulnerability scanner
- **OpenVAS**: Open-source vulnerability scanner
- **Qualys**: Cloud-based security platform
- **Rapid7 Nexpose**: Vulnerability management

## Incident Response Playbook

### Phase 1: Preparation
```python
class IncidentResponseTeam:
    def __init__(self):
        self.contacts = {
            'security_lead': 'security@company.com',
            'it_admin': 'admin@company.com',
            'legal': 'legal@company.com',
            'pr': 'pr@company.com'
        }
        self.tools_ready = False
        self.backups_verified = False
    
    def prepare(self):
        """Ensure team is ready for incidents"""
        self.verify_contact_list()
        self.check_tools()
        self.verify_backups()
        self.review_procedures()
```

### Phase 2: Detection & Analysis
```python
def analyze_incident(self, alert):
    """Analyze security alert"""
    incident = {
        'id': generate_incident_id(),
        'timestamp': datetime.now(),
        'severity': self.assess_severity(alert),
        'type': self.classify_incident(alert),
        'affected_systems': self.identify_affected_systems(alert),
        'indicators': self.extract_iocs(alert)
    }
    return incident
```

### Phase 3: Containment
```python
def contain_incident(self, incident):
    """Contain the security incident"""
    if incident['type'] == 'malware':
        self.isolate_infected_systems(incident['affected_systems'])
    elif incident['type'] == 'data_breach':
        self.revoke_compromised_credentials()
        self.block_exfiltration_channels()
    elif incident['type'] == 'ddos':
        self.enable_rate_limiting()
        self.activate_cdn_protection()
```

### Phase 4: Eradication
```python
def eradicate_threat(self, incident):
    """Remove threat from environment"""
    self.remove_malware()
    self.patch_vulnerabilities()
    self.reset_compromised_accounts()
    self.update_security_rules()
```

### Phase 5: Recovery
```python
def recover_systems(self, incident):
    """Restore normal operations"""
    self.restore_from_clean_backups()
    self.verify_system_integrity()
    self.monitor_for_reinfection()
    self.gradually_restore_services()
```

### Phase 6: Lessons Learned
```python
def post_incident_review(self, incident):
    """Document and learn from incident"""
    report = {
        'what_happened': incident['description'],
        'root_cause': self.determine_root_cause(incident),
        'response_effectiveness': self.evaluate_response(),
        'improvements': self.identify_improvements(),
        'updated_procedures': self.update_procedures()
    }
    return report
```

## Security Metrics & KPIs

```python
class SecurityMetrics:
    def __init__(self):
        self.metrics = {}
    
    def calculate_mttr(self, incidents):
        """Mean Time to Respond"""
        total_time = sum(i['response_time'] for i in incidents)
        return total_time / len(incidents) if incidents else 0
    
    def calculate_mttd(self, incidents):
        """Mean Time to Detect"""
        total_time = sum(i['detection_time'] for i in incidents)
        return total_time / len(incidents) if incidents else 0
    
    def vulnerability_remediation_rate(self, vulnerabilities):
        """Percentage of vulnerabilities fixed"""
        fixed = sum(1 for v in vulnerabilities if v['status'] == 'fixed')
        return (fixed / len(vulnerabilities)) * 100 if vulnerabilities else 0
    
    def security_awareness_score(self, phishing_tests):
        """Employee security awareness"""
        passed = sum(1 for t in phishing_tests if not t['clicked'])
        return (passed / len(phishing_tests)) * 100 if phishing_tests else 0
```

## Best Practices

1. **Continuous Monitoring**: Never stop watching your systems
2. **Defense in Depth**: Multiple layers of security controls
3. **Least Privilege**: Grant minimum necessary access
4. **Regular Updates**: Keep all systems patched
5. **Backup Strategy**: Regular, tested backups
6. **Documentation**: Maintain detailed procedures
7. **Training**: Regular security awareness training
8. **Threat Intelligence**: Stay informed about new threats
9. **Automation**: Automate repetitive security tasks
10. **Collaboration**: Work closely with Red Team for improvements

## Career Path

- **Security Analyst**: Monitor and analyze security events
- **Incident Responder**: Handle security incidents
- **Security Engineer**: Build and maintain security systems
- **SOC Analyst**: Security Operations Center monitoring
- **Threat Hunter**: Proactively search for threats
- **Security Architect**: Design security infrastructure
- **CISO**: Chief Information Security Officer

---

- Go back to [Cyber Security](./index.md)
- Return to [Home](../../index.md)
