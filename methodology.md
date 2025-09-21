methodology.md
# 🛠 Methodology – PCAP Analysis with tcpdump

This document explains, step by step, how I used **tcpdump** to analyze the packet capture (`tcpdump_challenge.pcap`).  
Each stage of analysis is supported by screenshots for clarity.

---

## 1. Verify tcpdump Installation
I started by confirming that tcpdump was installed and checking its version.

```bash
tcpdump --version


tcpdump v4.99.4

libpcap v1.10.4

OpenSSL 3.0.13

📸 Screenshot:

2. Count Packets in the Capture

To get an overview of the dataset size:

tcpdump -r tcpdump_challenge.pcap | wc -l


Total packets: 1344

📸 Screenshot:

3. Count ICMP Packets

Filtered only ICMP traffic to check for reconnaissance or ping activity:

tcpdump -r tcpdump_challenge.pcap icmp | wc -l


ICMP packets: 132

📸 Screenshot:

4. Inspect ICMP Echo Requests

Displayed the ICMP Echo Requests to identify source and destination:

tcpdump -r tcpdump_challenge.pcap 'icmp and icmp[icmptype] == icmp-echo'


Source: 10.0.2.10

Destination: 172.67.72.15

📸 Screenshot:

5. Attribute the Destination IP

Used whois to identify ownership of the destination IP.

whois 172.67.72.15 | grep -i 'origin\|descr'
whois -h whois.cymru.com "172.67.72.15"


ASN: 13335

Owner: Cloudflare (CLOUDFLARENET, US)

📸 Screenshot:

6. Inspect DNS and HTTP Traffic

Viewed ASCII payloads to check for DNS queries and HTTP requests.

tcpdump -r tcpdump_challenge.pcap -A


DNS queries for meter.net, huffpost.com, youtube.com

Responses resolved 172.67.72.15 (Cloudflare)

📸 Screenshot:

7. Confirm ICMP Replies

Validated that the Cloudflare server responded to the ICMP requests:

tcpdump -r tcpdump_challenge.pcap icmp


Observed ICMP echo reply messages.

📸 Screenshot:

8. Identify HTTP POST Requests

Searched the capture for HTTP POST traffic (often used for credentials).

tcpdump -r tcpdump_challenge.pcap -A | grep -i "post"
tcpdump -r tcpdump_challenge.pcap -A | grep -c "post"


Found 10 HTTP POST requests

Included traffic to www.huffpost.com

📸 Screenshot:

9. Extract Cleartext Credentials (HTTP)

Filtered for sensitive parameters like username and password.

tcpdump -nr tcpdump_challenge.pcap -A \
| egrep -i 'password=|user=|username=|Authorization: Basic'


Discovered:

Username: bsmith

Password: ilovecats9102

📸 Screenshot:

10. Analyze TCP Ports

Checked which TCP ports were most active.

tcpdump -nr tcpdump_challenge.pcap -nn tcp \
| awk '{for(i=1;i<=NF;i++){if($i==">"){split($(i-1),a,".");print a[length(a)];split($(i+1),b,".");print b[length(b)]}}}' \
| sort -n | uniq -c | sort -nr


Most traffic: HTTP (80)

Also present: FTP (21)

📸 Screenshot:

11. Extract FTP Credentials

Filtered traffic on port 21 for FTP login attempts.

tcpdump -nr tcpdump_challenge.pcap -nn -s0 -A port 21 | egrep -i "USER|PASS"


Cleartext credentials observed:

admin : admin

root : pass123

administrator : password

admin : password123

📸 Screenshot:

📸 Screenshot:

📸 Screenshot:

12. Inspect HTTP User-Agent Strings

Searched HTTP headers for User-Agent information.

tcpdump -nr tcpdump_challenge.pcap -s0 -A port 80 | grep -i "User-Agent"


Normal traffic: Chromium on Ubuntu

Suspicious: TeslaBrowser/5.5

Followed up to inspect TeslaBrowser traffic:

tcpdump -nr tcpdump_challenge.pcap -s0 -A port 80 | grep -A20 -B20 "TeslaBrowser/5.5"


Destination: t.me (Telegram)

Request path: /+zz0192Iskaaa

📸 Screenshot:

📸 Screenshot:

📸 Screenshot:

13. Confirm YouTube Activity

Verified benign activity (user streaming a YouTube video).

tcpdump -nr tcpdump_challenge.pcap -s0 -A port 80 | grep -i "youtube"


Host: www.youtube.com

Path: /watch?v=dQw4w9WgXcQ

📸 Screenshot: