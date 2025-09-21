methodology.md
🛠 Methodology – PCAP Analysis with tcpdump

This document explains how I used tcpdump to analyze the packet capture (tcpdump_challenge.pcap).
Each step includes the exact command, observation, and a matching screenshot.

1) Verify tcpdump Installation

Command

tcpdump --version


Observation

tcpdump v4.99.4

libpcap v1.10.4

OpenSSL 3.0.13

Screenshot


2) Count All Packets

Command

tcpdump -r tcpdump_challenge.pcap | wc -l


Observation

Total packets: 1344

Screenshot


3) Count ICMP Packets

Command

tcpdump -r tcpdump_challenge.pcap icmp | wc -l


Observation

ICMP packets: 132

Screenshot


4) Inspect ICMP Echo Requests

Command

tcpdump -r tcpdump_challenge.pcap 'icmp and icmp[icmptype] == icmp-echo'


Observation

Source: 10.0.2.10

Destination: 172.67.72.15

Screenshot


5) Attribute the Destination IP

Commands

whois 172.67.72.15 | grep -i 'origin\|descr'
whois -h whois.cymru.com "172.67.72.15"


Observation

ASN: 13335

Owner: Cloudflare (CLOUDFLARENET, US)

Screenshot


6) Inspect DNS & HTTP Payloads (ASCII)

Command

tcpdump -r tcpdump_challenge.pcap -A


Observation

DNS queries for meter.net, huffpost.com, youtube.com

Responses include Cloudflare IPs such as 172.67.72.15

Screenshot


7) Confirm ICMP Echo Replies

Command

tcpdump -r tcpdump_challenge.pcap icmp


Observation

ICMP echo replies observed from 172.67.72.15 to 10.0.2.10

Screenshot


8) Identify HTTP POST Requests

Commands

tcpdump -r tcpdump_challenge.pcap -A | grep -i "post"
tcpdump -r tcpdump_challenge.pcap -A | grep -c "post"


Observation

10 HTTP POSTs found

Includes traffic to www.huffpost.com

Screenshot


9) Extract Cleartext Credentials (HTTP)

Command

tcpdump -nr tcpdump_challenge.pcap -A | egrep -i 'password=|user=|username=|Authorization: Basic'


Observation

Credentials in POST body:

Username: bsmith

Password: ilovecats9102

Screenshot


10) Analyze Active TCP Ports

Command

tcpdump -nr tcpdump_challenge.pcap -nn tcp \
| awk '{for(i=1;i<=NF;i++){if($i==">"){split($(i-1),a,".");print a[length(a)];split($(i+1),b,".");print b[length(b)]}}}' \
| sort -n | uniq -c | sort -nr


Observation

Port 80 (HTTP) dominates

Port 21 (FTP) present → investigate

Screenshot


11) Extract FTP Credentials (Cleartext)

Command

tcpdump -nr tcpdump_challenge.pcap -nn -s0 -A port 21 | egrep -i "USER|PASS"


Observation

admin : admin

root : pass123

administrator : password

admin : password123

Screenshots






12) Inspect HTTP User-Agent Strings

Command

tcpdump -nr tcpdump_challenge.pcap -s0 -A port 80 | grep -i "User-Agent"


Observation

Normal UA: Chromium on Ubuntu

Suspicious UA: TeslaBrowser/5.5

Follow-up Command

tcpdump -nr tcpdump_challenge.pcap -s0 -A port 80 | grep -A20 -B20 "TeslaBrowser/5.5"


Observation

Destination: t.me (Telegram)

Path: /+zz0192Iskaaa → automation/C2-like

Screenshots






13) Confirm Benign YouTube Activity

Command

tcpdump -nr tcpdump_challenge.pcap -s0 -A port 80 | grep -i "youtube"


Observation

Host: www.youtube.com

Path: /watch?v=dQw4w9WgXcQ

Screenshot
