# 🔎 TCPDump PCAP Analysis – Real-World Case Study

This project is a **network forensics case study** where I used **tcpdump** to investigate a packet capture (`tcpdump_challenge.pcap`).  
The goal was to practice real SOC analyst workflows: **triage, evidence collection, findings, and security recommendations.**

All analysis was performed **only with tcpdump and Linux CLI tools** (no Wireshark GUI).  
This demonstrates my ability to perform **deep packet analysis directly from the terminal**.

---

## 📊 Executive Summary

- **Dataset**: `tcpdump_challenge.pcap`  
- **Environment**: Ubuntu VM, tcpdump v4.99.4  
- **Tools**: tcpdump, grep/awk/sed, whois  

### 🔐 Key Findings
- **ICMP Recon** → 132 echo requests to a Cloudflare IP (`172.67.72.15`).  
- **DNS Lookups** → Resolved domains: `meter.net`, `huffpost.com`, `youtube.com`.  
- **HTTP POSTs** → Found **cleartext login credentials**:  
  `bsmith : ilovecats9102`  
- **FTP Traffic** → Multiple weak credentials in plaintext:  
  `admin : admin`, `root : pass123`, `administrator : password123`  
- **Suspicious Behavior** → Custom User-Agent `TeslaBrowser/5.5` beaconing to **Telegram (`t.me`)**.  
- **Benign Context** → Normal browsing traffic to Google and YouTube (streaming a video).  

---

## 🛠️ Documentation

- [Methodology](methodology.md) → step-by-step tcpdump commands + workflow  
- [Findings](findings.md) → evidence and extracted results  
- [Recommendations](recommendations.md) → mitigations like a SOC report  

---

## 📸 Screenshots

| ICMP Recon | FTP Credentials | HTTP POST | Suspicious UA | YouTube |
|------------|----------------|-----------|---------------|---------|
| ![](screenshots/icmp.png) | ![](screenshots/ftp.png) | ![](screenshots/http.png) | ![](screenshots/teslabrowser.png) | ![](screenshots/youtube.png) |

---

## 🛡️ Recommendations (Summary)

- Replace **FTP** with **SFTP/FTPS** and disable weak/default accounts.  
- Enforce **HTTPS/TLS** for all web traffic (HSTS, TLS 1.2+).  
- Block/alert on **unusual User-Agents** (`TeslaBrowser/*`).  
- Apply **egress restrictions** to control traffic to Telegram (`t.me`).  
- Deploy IDS/IPS rules to flag **cleartext credentials** and **C2-like beaconing**.  

Full details → [recommendations.md](recommendations.md)

---

## ⚖️ Ethics

- All credentials and domains shown are **demo/lab values**.  
- No real user or customer data was exposed.  
- This repo is intended for **educational and portfolio purposes** only.  

---

## 👨‍💻 About Me

I’m Kwame A. Opuni, a Full-Stack Developer and Security Enthusiast with hands-on experience in:  

- **Blue Team Analysis** → packet captures, tcpdump, SOC investigations  
- **Development** → building tools in Node.js, React, PostgreSQL  
- **Cybersecurity** → CompTIA Security+, network monitoring, threat detection  

This repo is part of my portfolio to showcase how I can **analyze, document, and communicate technical findings like a SOC analyst.**

👉 If you’re a recruiter or hiring manager:  
Feel free to explore my work — I’d love to bring these skills to your team.  
