# 🦈 Wireshark & Network Analysis Lab

**Wireshark (Free) · Local Machine or Azure VM · Network Analysis**

---

## 📋 Lab Details

| Field | Value |
|---|---|
| **Certification Alignment** | CompTIA Network+ · Security+ · CySA+ |
| **Free Tools** | Wireshark — free and open source, no expiration, no account required |
| **Time to Complete** | 2–4 hours across multiple sessions |
| **Estimated Cost** | $0 — Wireshark is permanently free |
| **Career Relevance** | Network Engineer · SOC Analyst · Cloud Security Engineer · Incident Responder |

---

## 🏗️ Architecture — How Wireshark Captures Traffic

The diagram below shows how traffic flows from the internet through your network and into Wireshark. Understanding this flow is what makes everything in this lab click.

![Wireshark Capture Architecture](./diagrams/wireshark-architecture.svg)

---

## 💼 The Business Problem This Lab Solves

Networks carry every piece of data your organisation produces — emails, database queries, login credentials, file transfers, API calls. When something goes wrong — a service is unreachable, a user reports slow performance, a security alert fires — the network is almost always involved. The only way to know what is actually happening on a network is to look at the packets.

Wireshark is the tool organisations use to do that. It captures the raw data moving across a network interface and lets you inspect it at every layer — from the physical frame all the way up to the application payload.

| Role | How This Lab Applies |
|---|---|
| **Network Engineer** | Diagnose connectivity issues by seeing exactly where packets are dropped or delayed |
| **SOC Analyst** | Identify malicious traffic patterns, extract indicators of compromise from packet captures |
| **Cloud Security Engineer** | The mental model from Wireshark transfers directly to reading Azure Network Watcher and VPC flow logs |
| **Help Desk** | Prove that a reported network issue is real and identify whether it is client-side or server-side |

---

## 📖 Key Concepts — Read This Before Starting

These concepts will come up constantly throughout the lab. Read through them now so the steps make sense as you go.

### What is a Packet?

A packet is a small unit of data that travels across a network. When you send an email or load a web page, that data gets broken into hundreds or thousands of smaller packets. Each packet has a **header** — containing the source IP, destination IP, and port number — and a **payload**, which is the actual data. Packets travel independently across the network and get reassembled at the destination. Wireshark captures and shows you each individual packet.

### What is a Network Protocol?

A protocol is a set of rules that defines how data is formatted and transmitted across a network. Different protocols handle different jobs:
- **DNS** — translates domain names into IP addresses
- **HTTP** — transfers web page content
- **TCP** — ensures packets are delivered reliably
- **ICMP** — used for ping and network diagnostics

Each protocol has its own port number and packet structure. In Wireshark you can filter by protocol to see only the traffic you care about.

### What is the TCP Three-Way Handshake?

Before two computers can exchange data over TCP, they perform a three-step connection setup:

1. **SYN** — your machine sends: *"I want to connect."*
2. **SYN-ACK** — the server responds: *"I received your request, here is my acknowledgement."*
3. **ACK** — your machine confirms: *"Connection confirmed, ready to send data."*

If you see SYN but no SYN-ACK in a capture, the connection was refused or the server is unreachable. This is one of the most useful things to look for when diagnosing connectivity problems.

### What is DNS?

DNS is the system that translates human-readable domain names (like `google.com`) into IP addresses (like `142.250.80.46`) that computers use to communicate. Every time you visit a website, open an app, or send an email, a DNS query happens first. If DNS is broken, nothing works — users cannot reach websites, applications cannot connect to databases, email cannot be delivered.

### What is HTTP vs HTTPS?

**HTTP** is the protocol used to transfer web content. It is **unencrypted** — anyone who can see the network traffic can read every request and response, including usernames and passwords. **HTTPS** is HTTP with TLS encryption added on top. In this lab you will see cleartext credentials in an HTTP capture — that demonstration is exactly why HTTPS became the standard for every website that handles sensitive data.

### What is Promiscuous Mode?

Normally, a network interface card only captures packets addressed to your machine. In **promiscuous mode**, the NIC captures every single packet on the network segment — including packets addressed to other machines. Wireshark enables promiscuous mode automatically when you start a capture. On a switched network (most modern networks), you will primarily see your own traffic plus broadcast traffic.

---

## 🎯 What You Will Learn

| Skill | Real-World Application |
|---|---|
| Capture live network traffic | The foundational skill — you cannot analyse what you cannot see |
| Apply display filters | Production captures generate millions of packets. Filters let you find the 10 that matter in seconds |
| Read TCP handshakes | Recognising a normal three-way handshake versus an incomplete one tells you immediately whether a connection succeeded or failed |
| Identify DNS queries and responses | DNS is involved in virtually every network action. Seeing it in packets gives you a mental model that transfers to cloud DNS troubleshooting |
| Spot cleartext credentials in HTTP | A hands-on demonstration of why HTTPS matters — you will see actual credentials in plaintext |
| Follow TCP streams | Reassemble the full conversation between two hosts from individual packets — essential for incident investigation |

---

## 🚀 Step 1 — Get Wireshark

Wireshark is completely free and open source. Go to [wireshark.org/download.html](https://www.wireshark.org/download.html) and download the installer for your operating system. No account required, no trial, no licence.

| OS | Download | Notes |
|---|---|---|
| **Windows** | Windows x64 Installer (.exe) | Accept all defaults. Install **Npcap** when prompted — this is required to capture packets |
| **macOS** | macOS Arm or Intel (.dmg) | Run the installer. If prompted about ChmodBPF — allow it. This gives Wireshark permission to access network interfaces |
| **Linux** | Use your package manager | `sudo apt install wireshark` (Ubuntu/Debian). Add yourself to the `wireshark` group to capture without root |

```bash
# Linux only — add yourself to the wireshark group (log out and back in after)
sudo usermod -aG wireshark $USER

# Verify Wireshark is installed
wireshark --version
```

---

## 🖥️ Step 2 — Your First Capture

This step gets you comfortable with the Wireshark interface before doing anything complex.

1. Open Wireshark — on the welcome screen you will see a list of network interfaces with wavy lines showing live traffic activity
2. Double-click your active interface — **Ethernet** or **Wi-Fi**, pick the one with the most activity shown in the wave graph
3. Wireshark starts capturing immediately — packets appear in the list in real time
4. Open a browser and navigate to any website
5. After 30 seconds, click the **red square Stop** button in the toolbar

You now have a packet capture file in memory. The volume can be overwhelming — hundreds or thousands of packets from 30 seconds of browsing. This is exactly why display filters exist.

---

## 🔍 Step 3 — Essential Display Filters

Type filters into the filter bar at the top of the Wireshark window and press **Enter**. The packet list updates instantly to show only matching traffic.

### Display Filters vs Capture Filters

Wireshark has two types of filters:
- **Capture filters** — applied *before* capturing. They limit what gets recorded.
- **Display filters** — applied *after* capturing. They limit what you see without discarding anything.

For this lab, always use **display filters**. They let you go back and look at the same capture through different lenses without having to re-capture.

### Filter Reference

| Filter | What It Shows | When to Use It |
|---|---|---|
| `dns` | All DNS queries and responses | Troubleshooting name resolution, identifying unusual domain lookups |
| `http` | Unencrypted HTTP traffic only | Finding cleartext data, debugging web applications without HTTPS |
| `tcp` | All TCP traffic | Starting point for connectivity investigations |
| `tcp.flags.syn == 1` | TCP SYN packets only — connection attempts | Seeing which hosts are trying to connect to what |
| `tcp.flags.reset == 1` | TCP RST packets — connection resets | Finding refused or forcibly closed connections |
| `icmp` | All ICMP traffic including ping | Verifying basic reachability between hosts |
| `ip.addr == 192.168.1.1` | All traffic to or from a specific IP | Isolating traffic for a single host in a busy capture |
| `ip.src == 10.0.0.5` | Traffic from a specific source IP only | Isolating outbound traffic from one host |
| `tcp.port == 443` | All HTTPS traffic | Identifying encrypted web traffic by port |
| `http.request` | HTTP GET and POST requests only | Finding web requests — useful for spotting data exfiltration |

---

## 🧪 Step 4 — Guided Exercises

Work through each exercise in order. Each one builds on the previous and teaches a specific skill you will use in real-world network analysis.

---

### Exercise A — Capture a DNS Lookup

#### Before You Start: What is `nslookup` and Where Do You Run It?

`nslookup` is a built-in command-line tool that manually performs a DNS lookup. You run it on your **local machine — NOT inside Wireshark**. Wireshark is purely a packet capture and analysis tool. The workflow is: start a capture in Wireshark → switch to a terminal → run the command → return to Wireshark and stop the capture.

**How to open your terminal:**
- **Windows:** Press the Windows key, type `cmd`, press Enter
- **Mac:** Press `Cmd + Space`, type `Terminal`, press Enter
- **Linux:** Press `Ctrl + Alt + T`

#### What is a DNS A Record?

An **A record** (Address record) maps a domain name to an IPv4 address. When you type `google.com` into a browser, your computer makes a DNS query asking for the A record — and the response contains the IPv4 address to connect to. In Wireshark, the record type appears as a number — **type 1 is an A record**.

#### Steps

1. In Wireshark, start a capture on your active interface — click the **blue shark fin icon** or double-click your interface name
2. Leave Wireshark running and open a separate terminal window
3. In the terminal, run:
   ```
   nslookup google.com
   ```
4. The terminal will show the IP address(es) returned for `google.com` — this confirms the DNS lookup worked
5. Switch back to Wireshark and click the **red square Stop** button
6. In the filter bar, type `dns` and press Enter
7. Find the **query packet** — look in the Info column for `Standard query A google.com`
8. Find the **response packet** — look for `Standard query response A google.com`
9. Click the DNS response packet. In the packet detail pane, expand **Domain Name System (response)** → look inside the **Answers** section for the A record with the IP address. Confirm it matches what your terminal showed.

> ✅ What you just saw: your machine sent a DNS query asking for the A record for `google.com`. The DNS server replied with an IP address. Your browser then used that IP to make the actual connection. In the real world, unexpected DNS queries to unusual domains are often the first sign of malware calling home to a command-and-control server.

---

### Exercise B — Watch the TCP Three-Way Handshake

1. Start a capture on your active interface
2. Open a browser and navigate to `http://example.com` — use HTTP not HTTPS, this makes the handshake easier to see
3. Stop the capture
4. Run `nslookup example.com` first to get the IP address, then apply the filter: `tcp and ip.addr == [that IP]`
5. Find three packets in order: **SYN → SYN-ACK → ACK**

| Packet | Flags | What It Means |
|---|---|---|
| 1st packet | SYN | Your machine: *I want to connect. Here is my sequence number.* |
| 2nd packet | SYN, ACK | The server: *I got your request. Here is my sequence number. Connection accepted.* |
| 3rd packet | ACK | Your machine: *Got it. Connection is now open. Ready to send data.* |

> ✅ If you see a SYN but no SYN-ACK, the connection was refused or the server is unreachable. If you see a RST (reset) packet, the connection was forcibly closed. These two patterns are the most common things network engineers look for when diagnosing connectivity problems.

---

### Exercise C — Spot Cleartext Credentials (HTTP)

> ⚠️ **Educational exercise only.** Only capture on networks and against systems you own or have explicit permission to analyse. Never use this technique against systems you do not own.

1. Set up a test HTTP login form on your local machine or use a test site that runs over HTTP (not HTTPS)
2. Start a capture
3. Submit a login form over HTTP with a test username and password
4. Stop the capture
5. Apply the filter: `http.request.method == POST`
6. Click the POST packet and look in the packet detail pane — find the **HTML Form URL Encoded** layer
7. You will see the username and password in **plaintext**

> ✅ This is why every login form must use HTTPS. Without TLS encryption, anyone on the network path between you and the server — your ISP, a coffee shop router, anyone performing a man-in-the-middle attack — can read your credentials exactly as you typed them.

---

### Exercise D — Follow a Full TCP Stream

1. Capture any HTTP traffic by navigating to an HTTP website
2. Find any HTTP packet in the capture list
3. Right-click it → **Follow** → **TCP Stream**
4. Wireshark reassembles all the packets from that connection into a readable conversation
   - **Red text** = your browser's request
   - **Blue text** = the server's response

> ✅ This is how incident responders reconstruct what happened during a network event. Individual packets are fragments — the stream view shows the complete conversation, which is what you need to understand what data was transferred, what commands were sent, and what the server responded with.

---

## 💾 Step 5 — Save and Export Captures

Always save interesting captures. They are evidence of your skills and your portfolio entries for this lab.

```bash
# Save a capture for later analysis
File → Save As → choose .pcapng format

# Export only the packets matching your current filter
# Apply your display filter first, then:
File → Export Specified Packets → Displayed

# Re-open a saved capture
File → Open → select your .pcapng file
```

```bash
# Command-line capture with tshark (included with Wireshark — useful for remote servers)
tshark -i eth0 -w capture.pcapng -c 1000
# -i: interface name   -w: output file   -c: stop after this many packets
```

---

## ✅ Verification — Confirm the Lab is Working

| Skill | How to Verify |
|---|---|
| DNS capture | Apply `dns` filter and identify a query packet and its response packet — they should have matching transaction IDs |
| TCP handshake | Find three sequential packets with SYN, SYN-ACK, and ACK flags — explain what each one means without referring to notes |
| Display filters | Filter by IP address, port, and protocol from memory — you should not need to look these up |
| Stream reconstruction | Follow a TCP stream and read the full HTTP request and response as a conversation |
| File management | Save a capture, close Wireshark, reopen it, and load the file — confirm all packets are there |

---
