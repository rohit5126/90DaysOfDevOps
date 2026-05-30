# Day 14 – Networking Fundamentals & Hands-on Checks

## 7 layers of OSI Model

**Physical Layer-layer 1**

**Physical Layer which is your router, laptop, lan cable. these are part of physical layer*

⬇️

**Data link layer-layer 2**

**Data link layer which is physical addressing, MAC address(media access controller)*

⬇️

**Network layer-layer 3**

**Network layer requests through packets(ip Add)*

⬇️

**Transport layer-layer 4**

**Transport layer transports packet using protocol(TCP/IP, UDP)*

⬇️

**Session layer-layer 5**

**Session layer maintains session and establish connection*

⬇️

**Presentation layer-Layer 6**

**Presentation layer checks security, ehecks encryption, data encoding.*

⬇️

**Application layer-layer 7**

**Application layer is the direct intreface between user and network. HTTP, HTTPS, DNS*

## TCP/IP model

**Network access layer-1**

**it is combination of physical layer and data link layer*

⬇️

**Internet layer - layer2**

**its same as netwrok layer, requests data through packets*

⬇️

**Transport layer- layer3**

its same as transprot layer which sends packet using protocol*

⬇️

**application layer- layer4**

**this is combination of session, presentation and application layer as it consider establishing connection,

encryption and security and app interface to be a part of application.*


# Hands-on Checklist (run these; add 1–2 line observations)

hostname -I- this shows my IP.

ping <target> - this command shows the packets loss and if the url is working.

traceroute <target> (or tracepath) - to see the route of internet and how it reaches this site.

sudo ss -tulpn (or netstat -tulpn) - to see ports open and service running on them

dig <domain> or nslookup <domain> — provides the ip of the application server.

curl -I <http/https-url> — this shows the status code for the site. 200, 405.

netstat -an | head — shoes which port is listening.

