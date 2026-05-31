# Day 15 – Networking Concepts: DNS, IP, Subnets & Ports

## Task 1: DNS – How Names Become IPs.

**Explain in 3–4 lines: what happens when you type google.com in a browser?**

when we enter amazon.com in the browser it serches for any cache in browser or operating system if earlier visited the site. if it does not find any
it goes to the DNS resolver(one of the populer resolver is google(8.8.8.8)). DNS resolver send it to root server, root server finds its top level domain and sends it to TLD server,
TLD server sends it to specific server for Top level domain(.com,.in,.edu). TLd point it to authorization name server wher it gets the exact ip of the DNS and that goes to the 
user machine and user is able to access the website.


## What are these record types?

**The most common and foundational DNS record types include:**

A (Address) Record: The core record that points your domain or subdomain to a specific IPv4 address (e.g., 192.0.2.1).

AAAA (Quad-A) Record: Maps a hostname directly to a 128-bit IPv6 address (e.g., 2001:0db8::85a3) for newer network standards.

CNAME (Canonical Name) Record: Forwards an alias (e.g., www.example.com) to the actual canonical domain name instead of an IP address.

MX (Mail Exchange) Record: Directs incoming emails to the specific mail servers responsible for your domain's email delivery.

TXT (Text) Record: Stores arbitrary text information. These are widely used to verify domain ownership and for email security protocols like SPF, DKIM, and DMARC to prevent spam.

NS (Name Server) Record: Tells the internet which DNS servers hold the actual, authoritative records for a specific domain zone.

SOA (Start of Authority) Record: Stores crucial administrative information about a DNS zone, such as when it was last updated and the email address of the administrator.

SRV (Service) Record: Specifies the exact location (port and target host) for specific network services, such as VoIP or messaging.

## you can see TTL and record of any Domain using 'dig'

**dig trainwithshubham.com**

inside answer section

ANSWER SECTION:
trainwithshubham.com.   300     IN      A       15.197.225.128

trainwithshubham.com.   300     IN      A       3.33.251.168

[Domain name] -------| [TTL]  |----| [Record] |-----[IP]------|

## Task 2: IP Addressing

IPV4(internet protocol version 4) is a unique numerical identifier assigned to a device on a network.

Public IP is assigned to the server for accesssing the internet and connecting outside the network. it is shared publically.

Private Ip is assigned to devices to communicate within a local area network with other devices. it is kept hidden

this all are private IP ranges-
10.x.x.x, 
172.16.x.x – 172.31.x.x, 
192.168.x.x

## Task 3: CIDR & Subnetting

**/24 mean 192.168.1.0/24 - it means that there will be 2^(32-24) = 256 ip addresses.**

out of which first IP is always network IP and cannot be used.

and last one is broadcast ip which also cannot be used.

## How many usable hosts in a /24? A /16? A /28?

/24 = 254

/16 = 65534

/28 = 14

# Explain in your own words: why do we subnet?

we use subnet because there are only 4.3 billion ip address in the world. we decided to move to IPV6 but it is so complicated that
we decided to shift back to IPV4. but with a different approach that is using subnet.
we create a subnet for a private network, now each subnet can have 4.3 billion IPs.

| CIDR | Subnet Mask      | Total IPs | Usable Hosts |
|------|------------------|-----------|--------------|
| /24  | 255.255.255.0/24 | 256       | 254          |
| /16  | 255.255.0.0/16   | 65536     | 65534        |
| /28  | 255.255.255.240  | 16        | 14           |


## Task 4: Ports – The Doors to Services
A port is a virtual communication endpoint that allows devices to differentiate in network traffic.

| Port | Service |
|------|---------|
| 22   | SSH     |
| 80   | HTTP    |
| 443  | HTTPS   |
| 53   | DNS     |
| 3306 | Mysql   |
| 6379 | Redis   |
| 27017|Mongo DB |

# 5: Putting It Together

You run curl http://myapp.com:8080 — what networking concepts from today are involved? **TCP handhsake**

Your app can't reach a database at 10.0.1.50:3306 — what would you check first?
**first step to check is if the port is istening "nc -zv 10.0.1.50:3306"




