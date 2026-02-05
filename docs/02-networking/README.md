# Networking Fundamentals

[← Back to Index](../../README.md)

This section covers networking concepts essential for DevOps including OSI model, TCP/UDP, DNS, HTTP/HTTPS, firewalls, and troubleshooting.

---

<details>
<summary><strong>Q1: Explain the OSI model and its layers</strong></summary>

The OSI (Open Systems Interconnection) model is a conceptual framework with 7 layers that describes how network communication works.

| Layer | Name | Function | Protocols/Examples |
|-------|------|----------|-------------------|
| 7 | Application | User interface, application services | HTTP, FTP, SSH, DNS |
| 6 | Presentation | Data formatting, encryption | SSL/TLS, JPEG, ASCII |
| 5 | Session | Session management | NetBIOS, RPC |
| 4 | Transport | End-to-end delivery, reliability | TCP, UDP |
| 3 | Network | Routing, logical addressing | IP, ICMP, routers |
| 2 | Data Link | Physical addressing, framing | Ethernet, MAC, switches |
| 1 | Physical | Physical transmission | Cables, hubs, signals |

**Memory aid:** "All People Seem To Need Data Processing" (bottom-up)

**TCP/IP Model (simplified, practical):**

| TCP/IP Layer | OSI Layers |
|--------------|------------|
| Application | 7, 6, 5 |
| Transport | 4 |
| Internet | 3 |
| Network Access | 2, 1 |

**Data at each layer:**
- Application → Data
- Transport → Segment (TCP) / Datagram (UDP)
- Network → Packet
- Data Link → Frame
- Physical → Bits

</details>

<details>
<summary><strong>Q2: What is the difference between TCP and UDP?</strong></summary>

Both are Transport Layer (Layer 4) protocols but with different characteristics.

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented (3-way handshake) | Connectionless |
| **Reliability** | Guaranteed delivery, ordering | No guarantee |
| **Error checking** | Yes, with retransmission | Checksum only |
| **Speed** | Slower (overhead) | Faster |
| **Flow control** | Yes (windowing) | No |
| **Header size** | 20 bytes | 8 bytes |
| **Use cases** | HTTP, SSH, FTP, Email | DNS, DHCP, VoIP, Gaming, Streaming |

**TCP 3-Way Handshake:**
```
Client          Server
  |--- SYN ------->|
  |<-- SYN-ACK ----|
  |--- ACK ------->|
  |   Connection   |
  |  Established   |
```

**TCP Connection Termination (4-way):**
```
Client          Server
  |--- FIN ------->|
  |<-- ACK --------|
  |<-- FIN --------|
  |--- ACK ------->|
```

**When to use which:**
- **TCP**: When data integrity is critical (file transfer, web pages, database)
- **UDP**: When speed matters more than reliability (live video, gaming, DNS queries)

</details>

<details>
<summary><strong>Q3: Explain IP addressing and subnetting</strong></summary>

**IPv4 Address:**
- 32 bits, written as 4 octets (e.g., `192.168.1.100`)
- Each octet: 0-255

**IP Classes (historical):**

| Class | Range | Default Mask | Networks |
|-------|-------|--------------|----------|
| A | 1-126.x.x.x | /8 (255.0.0.0) | Large orgs |
| B | 128-191.x.x.x | /16 (255.255.0.0) | Medium orgs |
| C | 192-223.x.x.x | /24 (255.255.255.0) | Small orgs |

**Private IP Ranges (RFC 1918):**
```
10.0.0.0 - 10.255.255.255     (10.0.0.0/8)
172.16.0.0 - 172.31.255.255   (172.16.0.0/12)
192.168.0.0 - 192.168.255.255 (192.168.0.0/16)
```

**Subnetting Example:**
```
Network: 192.168.1.0/24
- Network address: 192.168.1.0
- Broadcast address: 192.168.1.255
- Usable hosts: 192.168.1.1 - 192.168.1.254 (254 hosts)

Subnet to /26 (255.255.255.192):
- 4 subnets, 62 hosts each
- 192.168.1.0/26   (hosts: .1-.62)
- 192.168.1.64/26  (hosts: .65-.126)
- 192.168.1.128/26 (hosts: .129-.190)
- 192.168.1.192/26 (hosts: .193-.254)
```

**CIDR Notation Quick Reference:**

| CIDR | Subnet Mask | Hosts |
|------|-------------|-------|
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /28 | 255.255.255.240 | 14 |
| /30 | 255.255.255.252 | 2 |

**Formula:** Hosts = 2^(32-CIDR) - 2

</details>

<details>
<summary><strong>Q4: What is DNS and how does it work?</strong></summary>

DNS (Domain Name System) translates domain names to IP addresses.

**DNS Resolution Process:**
```
1. User types: www.example.com

2. Browser → Local DNS Cache
   ↓ (not found)
3. OS → /etc/hosts file
   ↓ (not found)
4. OS → Recursive DNS Resolver (ISP or configured)
   ↓ (not cached)
5. Resolver → Root DNS Server (.)
   ↓ "Ask .com TLD server"
6. Resolver → TLD DNS Server (.com)
   ↓ "Ask example.com nameserver"
7. Resolver → Authoritative DNS (example.com)
   ↓ "IP is 93.184.216.34"
8. Response cached, returned to browser
```

**DNS Record Types:**

| Type | Purpose | Example |
|------|---------|---------|
| A | IPv4 address | `example.com → 93.184.216.34` |
| AAAA | IPv6 address | `example.com → 2606:2800:...` |
| CNAME | Alias to another domain | `www → example.com` |
| MX | Mail server | `mail.example.com (priority 10)` |
| TXT | Text records | SPF, DKIM, verification |
| NS | Nameserver | `ns1.example.com` |
| SOA | Start of Authority | Zone info, serial number |
| PTR | Reverse DNS | `34.216.184.93 → example.com` |

**DNS Commands:**
```bash
# Query DNS
nslookup example.com
dig example.com
dig +short example.com          # Just the IP
dig MX example.com              # Mail records
dig +trace example.com          # Full resolution path

# Reverse lookup
dig -x 8.8.8.8
```

**DNS Caching & TTL:**
- TTL (Time To Live) specifies how long records are cached
- Lower TTL = faster propagation, more DNS queries
- Higher TTL = slower propagation, less DNS load

</details>

<details>
<summary><strong>Q5: What is the difference between HTTP and HTTPS?</strong></summary>

| Feature | HTTP | HTTPS |
|---------|------|-------|
| **Port** | 80 | 443 |
| **Security** | Plain text | Encrypted (TLS/SSL) |
| **URL** | http:// | https:// |
| **Certificate** | Not required | Required |
| **SEO** | Lower ranking | Preferred by Google |

**HTTPS/TLS Handshake:**
```
Client                          Server
  |--- ClientHello ------------->|
  |<-- ServerHello + Cert -------|
  |--- Key Exchange ------------>|
  |<-- Finished -----------------|
  |   Encrypted Communication    |
```

**TLS Certificate Chain:**
```
Root CA (self-signed, trusted by browsers)
    └── Intermediate CA
            └── Server Certificate (your domain)
```

**Common HTTP Status Codes:**

| Code | Meaning |
|------|---------|
| 200 | OK |
| 201 | Created |
| 301 | Moved Permanently (redirect) |
| 302 | Found (temporary redirect) |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |
| 504 | Gateway Timeout |

**HTTP Methods:**

| Method | Purpose | Idempotent |
|--------|---------|------------|
| GET | Retrieve data | Yes |
| POST | Create resource | No |
| PUT | Update/replace resource | Yes |
| PATCH | Partial update | No |
| DELETE | Remove resource | Yes |
| HEAD | Get headers only | Yes |
| OPTIONS | Get allowed methods | Yes |

</details>

<details>
<summary><strong>Q6: Explain NAT (Network Address Translation)</strong></summary>

NAT translates private IPs to public IPs, allowing multiple devices to share one public IP.

**Types of NAT:**

| Type | Description |
|------|-------------|
| **SNAT (Source NAT)** | Changes source IP (outbound traffic) |
| **DNAT (Destination NAT)** | Changes destination IP (inbound traffic) |
| **PAT (Port Address Translation)** | Many private IPs → one public IP using ports |
| **Masquerade** | Like SNAT but auto-detects outbound IP |

**How PAT Works:**
```
Private Network              Router/NAT                Internet
192.168.1.10:50000  →  203.0.113.1:30001  →  Server
192.168.1.11:50000  →  203.0.113.1:30002  →  Server
192.168.1.12:50000  →  203.0.113.1:30003  →  Server
```

The router maintains a NAT table to map internal IPs/ports to external ports.

**Port Forwarding (DNAT):**
```
Internet Request          Router                Internal Server
203.0.113.1:80    →    NAT Rule    →    192.168.1.100:8080
```

**NAT in Linux (iptables):**
```bash
# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Masquerade (outbound)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Port forward (inbound)
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:8080
```

</details>

<details>
<summary><strong>Q7: What is DHCP and how does it work?</strong></summary>

DHCP (Dynamic Host Configuration Protocol) automatically assigns IP addresses to devices.

**DHCP Process (DORA):**
```
Client                    DHCP Server
  |--- DISCOVER (broadcast) -->|
  |<-- OFFER (IP offer) -------|
  |--- REQUEST (accept) ------>|
  |<-- ACK (confirmed) --------|
```

**What DHCP Provides:**
- IP address
- Subnet mask
- Default gateway
- DNS servers
- Lease time

**DHCP Lease:**
- Temporary IP assignment
- Client must renew before expiry
- Renewal at 50% of lease time, rebind at 87.5%

**Static vs Dynamic IP:**

| Static IP | Dynamic IP (DHCP) |
|-----------|-------------------|
| Manually configured | Automatically assigned |
| For servers, infrastructure | For clients, workstations |
| Never changes | May change on renewal |

**DHCP Relay:**
- DHCP uses broadcast (doesn't cross routers)
- DHCP relay forwards requests across subnets
- Configured on router/switch

**Commands:**
```bash
# View current DHCP lease
cat /var/lib/dhcp/dhclient.leases

# Release and renew
dhclient -r            # Release
dhclient               # Renew
```

</details>

<details>
<summary><strong>Q8: What is a VLAN and why use it?</strong></summary>

VLAN (Virtual Local Area Network) creates separate broadcast domains on the same physical switch.

**Why VLANs:**
- **Security**: Isolate sensitive traffic
- **Performance**: Reduce broadcast traffic
- **Organization**: Separate departments logically
- **Flexibility**: Move devices without rewiring

**Example:**
```
Physical Switch
├── VLAN 10 (Engineering) - Ports 1-8
├── VLAN 20 (HR) - Ports 9-16
└── VLAN 30 (Guest) - Ports 17-24
```

**VLAN Types:**

| Type | Description |
|------|-------------|
| Access Port | Belongs to one VLAN, for end devices |
| Trunk Port | Carries multiple VLANs, between switches |
| Native VLAN | Untagged traffic on trunk (default: VLAN 1) |

**802.1Q VLAN Tagging:**
- Trunk ports add 4-byte VLAN tag to frames
- Tag contains VLAN ID (1-4094)
- Access ports strip tags before delivery

**Inter-VLAN Routing:**
- VLANs are isolated by default
- Requires Layer 3 device (router or L3 switch)
- "Router on a stick" or SVI (Switch Virtual Interface)

**Linux VLAN Configuration:**
```bash
# Create VLAN interface
ip link add link eth0 name eth0.10 type vlan id 10
ip link set eth0.10 up
ip addr add 192.168.10.1/24 dev eth0.10
```

</details>

<details>
<summary><strong>Q9: Explain Load Balancing and its types</strong></summary>

Load balancing distributes traffic across multiple servers to improve availability and performance.

**Load Balancing Layers:**

| Layer | Name | Balances On |
|-------|------|-------------|
| L4 | Transport | IP + Port (TCP/UDP) |
| L7 | Application | HTTP headers, URL, cookies |

**L4 vs L7:**

| Feature | L4 | L7 |
|---------|----|----|
| Speed | Faster | Slower |
| Intelligence | Basic | Content-aware |
| SSL termination | No (passthrough) | Yes |
| Routing decisions | IP:Port | URL, headers, cookies |

**Load Balancing Algorithms:**

| Algorithm | Description |
|-----------|-------------|
| Round Robin | Rotate through servers sequentially |
| Weighted Round Robin | More traffic to higher-weight servers |
| Least Connections | Send to server with fewest connections |
| IP Hash | Same client IP always goes to same server |
| Least Response Time | Send to fastest responding server |

**Health Checks:**
- HTTP health check: `GET /health` → 200 OK
- TCP health check: Successful connection
- Failed checks → remove server from pool

**Common Load Balancers:**
- **HAProxy**: High-performance L4/L7
- **Nginx**: L7, also web server
- **AWS ELB/ALB/NLB**: Cloud-native
- **Traefik**: Container-native, auto-discovery

**HAProxy Example:**
```
frontend http_front
    bind *:80
    default_backend web_servers

backend web_servers
    balance roundrobin
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
    server web3 192.168.1.12:80 check
```

</details>

<details>
<summary><strong>Q10: What is a Firewall and how does it work?</strong></summary>

A firewall controls network traffic based on security rules.

**Firewall Types:**

| Type | Description |
|------|-------------|
| Packet Filter | Inspects headers (IP, port) |
| Stateful | Tracks connection state |
| Application (L7) | Inspects content (WAF) |
| Next-Gen (NGFW) | All of above + IDS/IPS |

**Stateless vs Stateful:**

| Stateless | Stateful |
|-----------|----------|
| Each packet independent | Tracks connection state |
| Needs rules for both directions | Return traffic auto-allowed |
| Faster, simpler | More secure, less rules |

**iptables Chains:**
```
                    ┌──────────────────┐
Incoming ──→ PREROUTING ──→ Routing ──→ FORWARD ──→ POSTROUTING ──→ Outgoing
                               │                          ↑
                               ↓                          │
                            INPUT ──→ Local ──→ OUTPUT ───┘
                                      Process
```

**iptables Example Rules:**
```bash
# Default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Block specific IP
iptables -A INPUT -s 192.168.1.100 -j DROP
```

**Cloud Security Groups:**
- AWS Security Groups = stateful firewall
- Rules allow, implicit deny
- Attached to instances/interfaces

</details>

<details>
<summary><strong>Q11: What are the differences between a Router, Switch, and Hub?</strong></summary>

| Device | OSI Layer | Function | Intelligence |
|--------|-----------|----------|--------------|
| Hub | Layer 1 | Repeats signals to all ports | None |
| Switch | Layer 2 | Forwards frames by MAC address | MAC table |
| Router | Layer 3 | Routes packets by IP address | Routing table |

**Hub (obsolete):**
- All devices receive all traffic
- Half-duplex, collisions
- No security, wastes bandwidth

**Switch:**
- Learns MAC addresses
- Forwards only to destination port
- Full-duplex, no collisions
- Creates separate collision domains

**Router:**
- Connects different networks
- Makes routing decisions
- NAT, firewall capabilities
- Separates broadcast domains

**Layer 3 Switch:**
- Combines switch + router functionality
- Routes between VLANs
- Hardware-accelerated routing

**Comparison:**

| Feature | Hub | Switch | Router |
|---------|-----|--------|--------|
| Broadcast domain | Same | Same | Separates |
| Collision domain | Same | Separates | Separates |
| Address used | None | MAC | IP |
| Speed | Slowest | Fast | Slower than L2 |

</details>

<details>
<summary><strong>Q12: Explain ARP (Address Resolution Protocol)</strong></summary>

ARP maps IP addresses to MAC addresses on a local network.

**How ARP Works:**
```
Device A wants to send to 192.168.1.20 (Device B)

1. A checks ARP cache for 192.168.1.20
   → Not found

2. A broadcasts ARP Request:
   "Who has 192.168.1.20? Tell 192.168.1.10"
   (Destination MAC: FF:FF:FF:FF:FF:FF)

3. All devices receive broadcast
   Only Device B responds

4. B sends ARP Reply (unicast):
   "192.168.1.20 is at AA:BB:CC:DD:EE:FF"

5. A caches the mapping and sends data
```

**ARP Commands:**
```bash
# View ARP cache
arp -a
ip neigh show

# Add static ARP entry
arp -s 192.168.1.20 AA:BB:CC:DD:EE:FF

# Clear ARP cache
ip neigh flush all
```

**ARP Table Entry:**
```
192.168.1.20  AA:BB:CC:DD:EE:FF  eth0
```

**ARP Security Issues:**
- **ARP Spoofing**: Attacker sends fake ARP replies
- **Man-in-the-Middle**: Intercept traffic
- **Mitigation**: Static ARP entries, Dynamic ARP Inspection

**Proxy ARP:**
- Router answers ARP requests on behalf of another network
- Allows communication between subnets without routing config

</details>

<details>
<summary><strong>Q13: What is the difference between IPv4 and IPv6?</strong></summary>

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Address size | 32 bits | 128 bits |
| Format | Dotted decimal (192.168.1.1) | Hexadecimal (2001:db8::1) |
| Total addresses | ~4.3 billion | 340 undecillion |
| Header | Variable, 20-60 bytes | Fixed, 40 bytes |
| Checksum | Yes | No (handled by L2/L4) |
| NAT | Commonly used | Not needed |
| IPSec | Optional | Built-in |
| Broadcast | Yes | No (multicast instead) |
| Configuration | DHCP or static | SLAAC or DHCPv6 |

**IPv6 Address Format:**
```
2001:0db8:0000:0000:0000:0000:0000:0001
     ↓ (remove leading zeros)
2001:db8:0:0:0:0:0:1
     ↓ (replace consecutive zeros with ::)
2001:db8::1
```

**IPv6 Address Types:**
- **Unicast**: One-to-one (global, link-local, loopback)
- **Multicast**: One-to-many (ff00::/8)
- **Anycast**: One-to-nearest

**Special Addresses:**
```
::1              - Loopback (like 127.0.0.1)
fe80::/10        - Link-local (auto-configured)
2000::/3         - Global unicast (public)
fc00::/7         - Unique local (like private IPv4)
```

**Check IPv6:**
```bash
ip -6 addr
ping6 google.com
```

</details>

<details>
<summary><strong>Q14: Explain common network troubleshooting commands</strong></summary>

**Connectivity Testing:**
```bash
# Basic connectivity
ping -c 4 host
ping -c 4 8.8.8.8          # Bypass DNS

# Trace route
traceroute host
mtr host                    # Continuous trace
```

**DNS Troubleshooting:**
```bash
nslookup domain.com
dig domain.com
dig +trace domain.com       # Full resolution path
host domain.com
```

**Port and Connection Testing:**
```bash
# Check if port is open
nc -zv host 80
telnet host 80

# Check listening ports
ss -tuln
netstat -tuln

# Check which process uses port
ss -tulnp
lsof -i :80
fuser 80/tcp
```

**Network Interface:**
```bash
ip addr                     # IP addresses
ip link                     # Interface status
ip route                    # Routing table
ifconfig                    # Legacy
```

**Packet Capture:**
```bash
# Capture traffic
tcpdump -i eth0
tcpdump -i eth0 port 80
tcpdump -i eth0 host 192.168.1.100
tcpdump -i eth0 -w capture.pcap    # Save to file

# Read capture file
tcpdump -r capture.pcap
wireshark capture.pcap      # GUI analysis
```

**Network Statistics:**
```bash
netstat -s                  # Protocol statistics
ss -s                       # Socket summary
ip -s link                  # Interface statistics
sar -n DEV 1                # Real-time throughput
```

**HTTP Testing:**
```bash
curl -v https://example.com
curl -I https://example.com  # Headers only
wget --spider https://example.com
```

</details>

<details>
<summary><strong>Q15: What is a Proxy Server and Reverse Proxy?</strong></summary>

**Forward Proxy:**
- Sits between clients and internet
- Clients configure to use proxy
- Hides client identity from servers

```
Clients → Forward Proxy → Internet → Servers
```

**Reverse Proxy:**
- Sits between internet and servers
- Clients don't know about it
- Hides server infrastructure

```
Clients → Internet → Reverse Proxy → Servers
```

**Comparison:**

| Feature | Forward Proxy | Reverse Proxy |
|---------|--------------|---------------|
| Protects | Clients | Servers |
| Client aware | Yes | No |
| Use cases | Corporate web filtering, anonymity | Load balancing, SSL termination |

**Reverse Proxy Benefits:**
- **Load balancing**: Distribute traffic
- **SSL termination**: Handle encryption
- **Caching**: Reduce backend load
- **Security**: Hide backend servers, WAF
- **Compression**: Reduce bandwidth

**Nginx as Reverse Proxy:**
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

</details>

<details>
<summary><strong>Q16: What ports should a DevOps engineer know?</strong></summary>

**Essential Ports:**

| Port | Protocol | Service |
|------|----------|---------|
| 20, 21 | TCP | FTP (data, control) |
| 22 | TCP | SSH, SFTP, SCP |
| 23 | TCP | Telnet (insecure) |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 67, 68 | UDP | DHCP (server, client) |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 123 | UDP | NTP |
| 143 | TCP | IMAP |
| 443 | TCP | HTTPS |
| 465 | TCP | SMTPS |
| 587 | TCP | SMTP (submission) |
| 993 | TCP | IMAPS |
| 995 | TCP | POP3S |

**Infrastructure Ports:**

| Port | Service |
|------|---------|
| 2379-2380 | etcd |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8080 | Common alt HTTP |
| 8443 | Common alt HTTPS |
| 9090 | Prometheus |
| 9200 | Elasticsearch |
| 27017 | MongoDB |

**Kubernetes Ports:**

| Port | Component |
|------|-----------|
| 6443 | API Server |
| 10250 | Kubelet |
| 10251 | Scheduler |
| 10252 | Controller Manager |
| 30000-32767 | NodePort range |

**Check if port is in use:**
```bash
ss -tuln | grep :80
lsof -i :80
netstat -tuln | grep :80
```

</details>

<details>
<summary><strong>Q17: Explain CDN (Content Delivery Network)</strong></summary>

A CDN caches content at edge locations worldwide to reduce latency.

**How CDN Works:**
```
User (Asia)               Origin Server (US)
     ↓
Edge Server (Asia) ← Cache or fetch
     ↓
Fast response
```

**CDN Benefits:**
- **Latency**: Content served from nearest edge
- **Bandwidth**: Offload origin server
- **DDoS protection**: Absorb attack traffic
- **Availability**: Multiple points of presence
- **SSL**: Edge SSL termination

**What CDN Caches:**

| Static (cached) | Dynamic (usually not cached) |
|-----------------|------------------------------|
| Images, CSS, JS | API responses |
| Videos | User-specific content |
| Documents | Shopping carts |
| Fonts | Real-time data |

**Cache Headers:**
```
Cache-Control: max-age=86400    # Cache for 1 day
Cache-Control: no-cache         # Revalidate before using
Cache-Control: no-store         # Never cache
ETag: "abc123"                  # Version identifier
```

**Popular CDNs:**
- CloudFlare
- AWS CloudFront
- Akamai
- Fastly
- Google Cloud CDN

**CDN Configuration Example (CloudFlare):**
- Proxy DNS through CloudFlare
- Set page rules for caching
- Configure security settings (WAF, DDoS)

</details>

<details>
<summary><strong>Q18: What is MTU and how does fragmentation work?</strong></summary>

**MTU (Maximum Transmission Unit):**
- Maximum packet size that can be sent without fragmentation
- Ethernet default: 1500 bytes
- Includes IP header (20 bytes) + data

**Path MTU Discovery:**
- Finds smallest MTU along entire path
- Uses "Don't Fragment" (DF) flag
- Receives ICMP "Fragmentation Needed" if too large

**Fragmentation:**
```
Original packet (3000 bytes)
        ↓
┌──────────────┐ ┌──────────────┐
│ Fragment 1   │ │ Fragment 2   │
│ 1500 bytes   │ │ 1500 bytes   │
│ offset=0     │ │ offset=1480  │
│ MF=1         │ │ MF=0         │
└──────────────┘ └──────────────┘
```

**Problems with Fragmentation:**
- Performance overhead
- If one fragment lost, entire packet retransmitted
- Security concerns (fragment attacks)

**Check MTU:**
```bash
# View interface MTU
ip link show eth0

# Test Path MTU
ping -M do -s 1472 destination   # 1472 + 28 (headers) = 1500

# Set MTU
ip link set eth0 mtu 1400
```

**Jumbo Frames:**
- MTU larger than 1500 (usually 9000)
- Reduces overhead for large transfers
- Must be supported end-to-end

</details>

<details>
<summary><strong>Q19: Explain TCP/IP connection states</strong></summary>

**Connection States:**

| State | Description |
|-------|-------------|
| LISTEN | Server waiting for connections |
| SYN_SENT | Client sent SYN, waiting for response |
| SYN_RECEIVED | Server received SYN, sent SYN-ACK |
| ESTABLISHED | Connection active |
| FIN_WAIT_1 | Sent FIN, waiting for ACK |
| FIN_WAIT_2 | Received ACK for FIN |
| CLOSE_WAIT | Received FIN, waiting to close |
| CLOSING | Both sides sent FIN simultaneously |
| LAST_ACK | Waiting for final ACK |
| TIME_WAIT | Waiting before reuse (2*MSL) |
| CLOSED | Connection terminated |

**Normal Connection Lifecycle:**
```
LISTEN → SYN_RECEIVED → ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED
         (server)

CLOSED → SYN_SENT → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED
         (client)
```

**View Connection States:**
```bash
ss -tan state established
ss -tan state time-wait
netstat -an | grep ESTABLISHED | wc -l
```

**TIME_WAIT:**
- Lasts 2*MSL (Maximum Segment Lifetime), typically 60 seconds
- Ensures late packets don't affect new connections
- Can cause port exhaustion under high load

**Tuning:**
```bash
# Allow reuse of TIME_WAIT sockets
net.ipv4.tcp_tw_reuse = 1
```

</details>

<details>
<summary><strong>Q20: What is the difference between Layer 2 and Layer 3 networking?</strong></summary>

**Layer 2 (Data Link):**
- Uses MAC addresses
- Local network communication
- Switches operate here
- Broadcast domain = all connected devices

**Layer 3 (Network):**
- Uses IP addresses
- Routes between networks
- Routers operate here
- Enables internet connectivity

**Comparison:**

| Aspect | Layer 2 | Layer 3 |
|--------|---------|---------|
| Address | MAC (48-bit) | IP (32/128-bit) |
| Scope | Local LAN | Global/Internet |
| Device | Switch | Router |
| Protocol | Ethernet | IP |
| Broadcast | Within segment | Blocked by router |

**When to use each:**

**Layer 2 (switching):**
- Same subnet communication
- High-speed, low latency
- Simple, no routing needed

**Layer 3 (routing):**
- Cross-subnet communication
- Internet connectivity
- Network segmentation
- Access control between segments

**Layer 3 Switch:**
- Combines both capabilities
- Routes at hardware speed
- Used for inter-VLAN routing

**Practical Example:**
```
VLAN 10 (192.168.10.0/24)           VLAN 20 (192.168.20.0/24)
├── PC1 (192.168.10.10)             ├── PC3 (192.168.20.10)
├── PC2 (192.168.10.11)             ├── PC4 (192.168.20.11)
        ↓                                    ↓
    Layer 2 switch ←──── Layer 3 router ────→ Layer 2 switch
    (within VLAN)         (between VLANs)      (within VLAN)
```

</details>

