# DevOps Journey — Daily Notes

---

# Day 01 — DevOps & Cloud Introduction + Linux Networking

## 📚 Module 01 — DevOps & DevSecOps Introduction

### What is DevOps?
- Culture + mindset that bridges Development and Operations teams
- Before DevOps: Dev team wrote code → threw it to Ops → Ops deployed → blame game when it broke
- After DevOps: Both teams own the entire lifecycle together
- Core pillars: Collaboration, Automation, Fast delivery, Continuous improvement

### What is DevSecOps?
- DevOps + Security embedded at every stage
- "Shift-left security" — catch vulnerabilities EARLY in development, not after deployment
- Security is everyone's responsibility, not just a security team at the end
- Security as code — automated scanning in every pipeline

### DevOps vs DevSecOps
DevOps:    Dev → Test → Deploy → Monitor
DevSecOps: Dev → Scan → Test → Scan → Deploy → Monitor → Scan
↑              ↑                         ↑
code analysis   dependency check          runtime check

### Benefits I need to remember for interviews
- Faster deployments with fewer failures
- Early vulnerability detection = cheaper to fix
- Shared ownership = no blame culture
- Compliance readiness built in

### Evolution of Software Delivery
Traditional SDLC → Agile → CI/CD → DevOps → DevSecOps
(months)     (weeks)  (days)  (hours)   (minutes)

### Real-Time Corporate DevOps Workflow
Developer commits code
↓
CI triggered via webhook (GitHub → Jenkins/GitHub Actions)
↓
Build (Maven/NPM/pip)
↓
Code Quality Scan (SonarQube)
↓
Security Scan (Trivy/OWASP)
↓
Artifact created → pushed to Nexus
↓
Docker image built → pushed to ECR/DockerHub
↓
Deploy to Kubernetes (EKS/AKS)
↓
Monitor (Prometheus + Grafana)

### Tools at each stage (memorize this for interviews)
| Stage | Tools |
|-------|-------|
| Source Control | Git, GitHub, GitLab |
| CI/CD | Jenkins, GitHub Actions, GitLab CI |
| Code Quality | SonarQube |
| Security | Trivy, OWASP, Gitleaks, Checkov |
| Artifacts | Nexus, AWS ECR |
| Containers | Docker |
| Orchestration | Kubernetes (EKS, AKS) |
| IaC | Terraform, Ansible |
| Monitoring | Prometheus, Grafana, ELK |

### Deployment Strategies (asked in every DevOps interview)
Blue-Green:  Two identical environments
Traffic switches instantly from Blue to Green
Instant rollback = switch back
Zero downtime ✅
Rolling:     Update pods gradually one by one
Old and new versions run simultaneously during rollout
Zero downtime ✅
Canary:      Release to 5% of users first
Monitor for errors
If healthy → roll out to 100%
Safest for high-risk releases ✅
Recreate:    Stop old version completely → start new version
Has downtime ❌
Used for non-critical apps only

### DevOps Team Structure
- Pipeline ownership → DevOps engineers
- Infrastructure management → Cloud/Infra engineers  
- Release coordination → Release managers
- Security enforcement → DevSecOps engineers
- All teams share accountability — no silos

---

## 📚 Module 02 — Linux Networking (Day 01)

### How TCP/IP works
- IP = address on the envelope (which machine)
- TCP = guarantee all packets arrive in order (nothing missing)
- Data breaks into packets → travels through routers → reassembled at destination
- TTL (Time To Live) — decrements at each hop, prevents infinite loops

### My traceroute to Google (11 hops from Pakistan)
Hop 1  172.24.16.1      → WSL virtual gateway
Hop 2  192.168.10.1     → My home WiFi router
Hop 3  119.156.176.1    → Pakistan ISP (PTCL)
Hop 4-6  10.253.x.x    → ISP internal backbone
Hop 7  74.125.118.170  → Entered Google's network 🎯
Hop 8  * * *           → Google firewall (blocked traceroute — normal)
Hop 11 142.250.202.142 → Google's server ✅
Total: 35ms latency

### DNS — How domain names resolve to IPs
Full resolution chain (dig +trace showed this live):
Root servers (13 globally) → "for .com ask Verisign"
.com servers (Verisign)    → "for google.com ask Google's NS"
Google nameservers         → "IP is 142.250.202.142"
Cached = Non-authoritative answer (faster)
Fresh  = Authoritative answer (slower, goes full chain)

### DNS Record Types
| Record | Purpose | DevOps Use |
|--------|---------|------------|
| A | domain → IPv4 | Point domain to server IP |
| AAAA | domain → IPv6 | IPv6 routing |
| CNAME | domain → domain | Aliases, load balancers |
| MX | mail server | Jenkins email notifications |
| NS | nameserver | Domain delegation |
| TXT | text data | SSL verification, SPF |

### TTL — Time To Live
- DNS records have TTL in seconds
- `google.com 300 IN A 142.250.202.142` → cached for 300 seconds
- **Before server migration:** lower TTL to 60s so changes propagate fast
- **Normal operation:** high TTL (3600s) reduces DNS load

### /etc/hosts — Local DNS override
- Checked BEFORE any DNS server
- Used to test Kubernetes Ingress locally
- Format: `IP   hostname`
- Example I tested: `echo "1.2.3.4 testapp.local" >> /etc/hosts`
- Production use: never — only for local testing

### /etc/resolv.conf — DNS resolver config
- `nameserver 10.255.255.254` → WSL's DNS resolver
- In Kubernetes: auto-injected into every pod by CoreDNS
- That's how `http://my-service` resolves inside a cluster

### TLS/HTTPS — How encryption works
Problem: HTTP sends data as plain text — anyone can read it
Solution: HTTPS encrypts everything using TLS
TLS 1.3 Handshake (what I saw live with curl -v):

Client Hello  → "I want TLS 1.3, here are my cipher suites"
Server Hello  → "OK, let's use TLS_AES_256_GCM_SHA384"
Certificate   → Server sends identity proof (4869 bytes for Google)
CERT verify   → Proves server owns the certificate
Finished      → Both sides ready, tunnel established
Encrypted     → All data from here is unreadable to anyone else


### Certificate Chain of Trust (what I saw with openssl)
GlobalSign Root CA (pre-installed in your browser/OS — ultimate trust)
↓ signed
Google Trust Services WR2 (intermediate CA)
↓ signed
*.google.com (the actual certificate — valid May→Aug 2026, 90 days)
- Wildcard cert `*.google.com` covers ALL subdomains
- In Month 3: I'll use Let's Encrypt as CA for my Kubernetes apps (free, auto-renews)

### Well-known ports (memorize for interviews)
| Port | Service |
|------|---------|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 8080 | Jenkins |
| 9000 | SonarQube |
| 8081 | Nexus |
| 3000 | Grafana |
| 9090 | Prometheus |
| 5432 | PostgreSQL |
| 3306 | MySQL |
| 27017 | MongoDB |
| 6443 | Kubernetes API |
| 2379 | etcd |

### Commands I ran today
```bash
ping google.com -c 4              # Test connectivity, check latency
traceroute google.com             # See every hop to destination
nslookup google.com               # Basic DNS lookup
dig google.com                    # Detailed DNS query with TTL
dig google.com +trace             # Full DNS resolution chain
dig google.com MX                 # Mail exchange records
cat /etc/hosts                    # Local DNS overrides
cat /etc/resolv.conf              # DNS resolver config
echo "1.2.3.4 testapp.local" | sudo tee -a /etc/hosts  # Add local DNS
sudo sed -i '/testapp.local/d' /etc/hosts               # Remove it
curl -v https://google.com        # See full TLS handshake
echo | openssl s_client -connect google.com:443         # See certificate chain
echo | openssl s_client -connect google.com:443 | grep "NotAfter"  # Check expiry
```

### Tomorrow — Day 02
- Ports deep dive: ss, netstat, nc
- UFW firewall rules
- Namespaces & Cgroups (Docker internals)
- First shell script