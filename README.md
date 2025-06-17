# THM-TCDUMP
Command-line packet forensics using Tcpdump to filter, inspect, and analyze network traffic in PCAP files for protocol-specific insights and anomaly detection.

By Ramyar Daneshgar 

### **Task 2: Basic Packet Capture**

> **What option can you add to your command to display addresses only in numeric format?**

When I began inspecting traffic with Tcpdump, I noticed it was attempting to resolve IPs to hostnames and port numbers to service names. This was introducing noise and latency during analysis. To eliminate DNS lookups and work directly with raw IP addresses and port values—which is essential for forensic accuracy—I added the `-n` flag:

```bash
tcpdump -n
```

This ensured I was seeing clean, numeric output with no external resolution involved, a best practice in threat hunting scenarios.

**Answer:** `-n`

---

### **Task 3: Filtering Expressions**

> **How many packets in traffic.pcap use the ICMP protocol?**

ICMP traffic often reflects scanning activity, misconfigurations, or network diagnostics. To identify all ICMP packets, I read the capture file using the `-r` flag and filtered by protocol. Then I counted the results using `wc -l`:

```bash
tcpdump -r traffic.pcap icmp | wc -l
```

Each line represented one ICMP packet. This gave me a precise packet count without loading the capture into a GUI tool.

**Answer:** `26`

---

> **What is the IP address of the host that asked for the MAC address of 192.168.124.137?**

I wanted to find which device sent an ARP request for the IP 192.168.124.137. ARP is a Layer 2 protocol used to resolve IP-to-MAC mappings, and such requests are usually part of network discovery or standard communication prep.

I ran:

```bash
tcpdump -r traffic.pcap arp
```

Then I parsed the output for "who-has 192.168.124.137". The line included a “tell” directive, pointing to the source IP of the request:

```
Who has 192.168.124.137? Tell 192.168.124.148
```

This told me that `.148` was the initiator of the ARP query.

**Answer:** `192.168.124.148`

---

> **What hostname (subdomain) appears in the first DNS query?**

To determine the first domain name queried, I filtered DNS packets by isolating port 53 traffic. I used `head` to limit the output and quickly inspect the first few DNS-related packets:

```bash
tcpdump -r traffic.pcap port 53 | head -3
```

I scanned the initial DNS queries and found the QNAME field pointing to:

**Answer:** `mirrors.rockylinux.org`

---

### **Task 4: Advanced Filtering**

> **How many packets have only the TCP Reset (RST) flag set?**

RST packets are a red flag when seen in high volumes—they often appear during service enumeration, scan rejection, or failed session handshakes. Tcpdump doesn’t directly expose symbolic TCP flag filters, so I had to read through the TCP traffic and look for “RST” flags in the output manually.

After reviewing the filtered results, I confirmed that **57** packets matched the pattern of having only the RST flag set.

**Answer:** `57`

---

> **What is the IP address of the host that sent packets larger than 15000 bytes?**

Packets over 1500 bytes exceed the typical MTU and often represent either file transfers or abuse scenarios like fragmentation attacks or tunneling. I used the `greater` keyword to find oversized traffic:

```bash
tcpdump -r traffic.pcap -n 'greater 15000'
```

This output revealed a packet over 15,000 bytes in length, with a source IP of:

**Answer:** `185.117.80.53`

---

### **Task 5: Displaying Packets**

> **What is the MAC address of the host that sent an ARP request?**

MAC addresses are only visible when Ethernet headers are included. By default, Tcpdump omits Layer 2 info. To expose this, I added the `-e` flag:

```bash
tcpdump -r traffic.pcap -e arp
```

This allowed me to inspect the source MAC address in the ARP frame. I found the "who-has" packet that initiated the request and extracted the originating MAC address from the frame’s Ethernet header:

**Answer:** `52:54:00:7c:d3:5b`

---

## **Lessons Learned**

1. **Tcpdump offers unmatched speed and control for packet-level forensics**, especially when analyzing specific protocol behavior or filtering large PCAPs under time constraints.
2. **Using flags like `-n` and `-e` avoids abstraction**, giving me access to clean numeric data and Layer 2 headers—essential when assessing real-world anomalies or rogue devices.
3. **Combining Tcpdump with shell tools like `wc`, `head`, and `grep` allowed me to answer forensic questions without needing to parse the entire capture manually.**
4. **Protocol filtering is surgical when applied correctly**—targeting only ICMP, ARP, DNS, or RST traffic enabled rapid hypothesis testing during incident triage.
5. **I reinforced the importance of understanding Layer 2–4 interactions**, such as how ARP ties into IP communications and how TCP control flags signal session state or hostile activity.
