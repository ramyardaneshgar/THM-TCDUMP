# THM-TCDUMP
Command-line packet forensics using Tcpdump to filter, inspect, and analyze network traffic in PCAP files for protocol-specific insights and anomaly detection.

By Ramyar Daneshgar 

### **Task 2: Basic Packet Capture**

> **What option can you add to your command to display addresses only in numeric format?**

By default, Tcpdump attempts to resolve IP addresses to hostnames and port numbers to service names using DNS and `/etc/services`. This behavior can introduce unnecessary resolution delays and clutter the raw data with potentially misleading aliases, especially in environments where DNS spoofing is a risk.

Adding the `-n` flag tells Tcpdump to **suppress all name resolution**, allowing us to work directly with numerical IPs and ports. This is especially important during packet analysis for intrusion detection or threat hunting, where IP integrity and performance are key.

**Answer:** `-n`

---

### **Task 3: Filtering Expressions**

> **How many packets in traffic.pcap use the ICMP protocol?**

ICMP (Internet Control Message Protocol) is used primarily for diagnostics and control messages, such as ping requests or destination unreachable errors. Using Tcpdump’s read mode (`-r`), we can replay the capture file and apply a protocol filter:

```bash
tcpdump -r traffic.pcap icmp | wc -l
```

This command filters for ICMP protocol packets and uses `wc -l` to count the number of lines output, each corresponding to one packet. This provides a quantitative metric of ICMP traffic within the capture.

**Answer:** `26`

---

> **What is the IP address of the host that asked for the MAC address of 192.168.124.137?**

This involves inspecting ARP traffic. ARP (Address Resolution Protocol) requests are used to discover the MAC address corresponding to an IP on a local subnet. In the ARP “who-has” packet, the sender IP is the one performing the query.

Using:

```bash
tcpdump -r traffic.pcap arp
```

We read through the ARP packets and found a request targeting 192.168.124.137. The packet format typically reads:
`who has 192.168.124.137 tell 192.168.124.148`
This shows that 192.168.124.148 is the device issuing the request.

**Answer:** `192.168.124.148`

---

> **What hostname (subdomain) appears in the first DNS query?**

DNS queries are sent over port 53 using UDP or TCP depending on payload size. The goal here is to identify the first domain queried, which provides insights into the initial destination of outbound connections.

```bash
tcpdump -r traffic.pcap port 53 | head -3
```

This filters DNS traffic and previews the first few packets. Within the output, the queried domain appears in the QNAME field. The first request in the capture resolves the domain:

**Answer:** `mirrors.rockylinux.org`

---

### **Task 4: Advanced Filtering**

> **How many packets have only the TCP Reset (RST) flag set?**

TCP Reset (RST) flags are used to abruptly terminate a connection. A surge in RST packets may indicate scanning activity, dropped sessions, or firewall behavior rejecting connections.

Tcpdump doesn’t support symbolic filtering like `tcp.flags == RST`, so we typically analyze this through either verbose output parsing or filtering using byte-offset expressions. Alternatively, we extract relevant packets using broader TCP filters and post-process via grep or scripting.

From the room output, the total count provided was:

**Answer:** `57`

---

> **What is the IP address of the host that sent packets larger than 15000 bytes?**

To detect jumbo packets (unusually large frames), we use the `greater` keyword in Tcpdump, which filters packets by size. Large packets may be indicative of file transfers, attacks (like fragmentation-based evasion), or MTU misconfiguration.

```bash
tcpdump -r traffic.pcap -n 'greater 15000'
```

This returns only packets exceeding 15,000 bytes in length. By disabling name resolution (`-n`), we ensure raw IP visibility. The source IP from the output was:

**Answer:** `185.117.80.53`

---

### **Task 5: Displaying Packets**

> **What is the MAC address of the host that sent an ARP request?**

To inspect MAC addresses, we must include Ethernet-level headers in Tcpdump output using the `-e` flag. This reveals the source and destination MAC addresses for each frame, which are critical for mapping host identities at Layer 2.

```bash
tcpdump -r traffic.pcap -e arp
```

Scanning the output, we locate the ARP “who-has” request and identify the associated source MAC address from the Ethernet frame header. This MAC corresponds to the interface of the querying device:

**Answer:** `52:54:00:7c:d3:5b`

---

## **Lessons Learned**

1. **Tcpdump remains one of the most efficient tools for packet-level forensic analysis** on the command line, especially for triaging incidents when access to full-featured GUI tools like Wireshark is limited or impractical.

2. **Protocol filters, size constraints, and display flags allow surgical packet selection**, reducing noise and enabling faster threat identification during incident response or malware beacon analysis.

3. **Layer 2 through Layer 7 data can be selectively exposed**, depending on flags like `-e`, `-X`, and `-A`, reinforcing the importance of understanding which layers are involved for each task.

4. **Disabling name resolution with `-n` ensures operational speed and integrity**, especially in adversarial or internal DNS spoofing scenarios where domain lookups can be misleading or manipulated.

5. **Reading `.pcap` files with specific filters replicates what a Security Operations Center (SOC) analyst would do** in a real investigation—extracting key indicators, scanning for anomalies, and tracing endpoint behavior based on protocol activity.

6. **Mastering Tcpdump is foundational for blue teamers, network defenders, and forensic analysts**, and this room reinforced not just the syntax, but the investigative mindset necessary to identify and interpret relevant traffic patterns quickly.

