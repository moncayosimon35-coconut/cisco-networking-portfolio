# IPv6 Network Project — Cisco Packet Tracer

## 1. Project overview

This project builds an IPv6 network in Cisco Packet Tracer using four Cisco 2911 routers, five LANs, switches, six PCs, and two servers. The goal is to provide connectivity between the LANs and make a website accessible by its domain name.

The four routers are:

- **ANTARTIDA** — central router connecting the other three routers and hosting the DNS server LAN.
- **ASIA** — provides two separate LANs for four PCs.
- **COLOMBIA** — provides one LAN for two PCs.
- **EUROPA** — hosts the web server LAN.

The design uses **IPv6 static routing** between routers. The ASIA and COLOMBIA LANs use DHCPv6 configuration for PCs. The DNS and web server addresses are configured manually.

> **Current verified milestone:** all four routers' relevant interfaces were shown as `up/up`; PCs successfully pinged their local gateways; after the missing static routes were added on ANTARTIDA, pings from an ASIA PC to both server IPv6 addresses succeeded. DNS record resolution and HTTP page access should be tested after configuring the DNS and HTTP services.

---

## 2. Topology and interconnections

ANTARTIDA is the central router. It has one Ethernet connection to the DNS server LAN and three serial links to ASIA, COLOMBIA, and EUROPA.

| From | Interface | To | Interface | Purpose |
|---|---|---|---|---|
| ANTARTIDA | `G0/1` | DNS LAN switch | Switch port | DNS server LAN |
| ANTARTIDA | `S0/0/0` | ASIA | `S0/0/0` | Router-to-router link |
| ANTARTIDA | `S0/0/1` | COLOMBIA | `S0/0/0` | Router-to-router link |
| ANTARTIDA | `S0/1/0` | EUROPA | `S0/0/0` | Router-to-router link |
| ASIA | `G0/1` | ASIA LAN 1 switch | Switch port | LAN 1; two PCs |
| ASIA | `G0/2` | ASIA LAN 2 switch | Switch port | LAN 2; two PCs |
| COLOMBIA | `G0/1` | COLOMBIA LAN switch | Switch port | LAN 3; two PCs |
| EUROPA | `G0/1` | Web LAN switch | Switch port | Web server LAN |

*Interface names are abbreviated: `G` = GigabitEthernet; `S` = Serial.*

### Logical topology

```text
                         [DNS Server]
                              |
                          [Switch]
                              |
                    G0/1 — ANTARTIDA
                         /    |     \
                        /     |      \
                   Serial  Serial   Serial
                     /        |        \
                  ASIA     COLOMBIA    EUROPA
                 /   \         |          |
              G0/1   G0/2     G0/1       G0/1
               |      |        |          |
            [SW]    [SW]     [SW]        [SW]
             / \     / \      / \          |
           PC  PC  PC  PC   PC  PC     [Web Server]
```

---

## 3. IPv6 addressing plan

Every LAN and router-to-router link uses a `/64` prefix. The prefix identifies the network; the remaining address bits identify an interface or host.

### 3.1 The five LANs

| LAN | Router/interface | Network prefix | Device addresses |
|---|---|---|---|
| DNS LAN | ANTARTIDA `G0/1` | `2001:b0ba:fea:1::/64` | DNS server: `2001:b0ba:fea:1::10/64`; router uses EUI-64 |
| Web LAN | EUROPA `G0/1` | `2001:b0ba:fea:2::/64` | Web server: `2001:b0ba:fea:2::10/64`; router uses EUI-64 |
| ASIA LAN 1 | ASIA `G0/1` | `2001:beba:cafe:1::/64` | Gateway: `2001:beba:cafe:1::1`; PCs use DHCPv6 |
| ASIA LAN 2 | ASIA `G0/2` | `2001:beba:cafe:2::/64` | Gateway: `2001:beba:cafe:2::1`; PCs use DHCPv6 |
| COLOMBIA LAN 3 | COLOMBIA `G0/1` | `2001:beba:cafe:3::/64` | Gateway: `2001:beba:cafe:3::1`; PCs use DHCPv6 |

### 3.2 Router-to-router serial links

These links are separate IPv6 networks; they are not counted among the five LANs.

| Link | ANTARTIDA interface/address | Remote interface/address | Prefix |
|---|---|---|---|
| ANTARTIDA–ASIA | `S0/0/0` — `2001:b0ba:fea:10::1/64` | ASIA `S0/0/0` — `2001:b0ba:fea:10::2/64` | `2001:b0ba:fea:10::/64` |
| ANTARTIDA–COLOMBIA | `S0/0/1` — `2001:b0ba:fea:11::1/64` | COLOMBIA `S0/0/0` — `2001:b0ba:fea:11::2/64` | `2001:b0ba:fea:11::/64` |
| ANTARTIDA–EUROPA | `S0/1/0` — `2001:b0ba:fea:12::1/64` | EUROPA `S0/0/0` — `2001:b0ba:fea:12::2/64` | `2001:b0ba:fea:12::/64` |

### 3.3 Verified EUI-64 router addresses

EUI-64 creates an interface identifier from the interface MAC address while preserving the network prefix.

| Router interface | IPv6 address |
|---|---|
| ANTARTIDA `G0/1` | `2001:b0ba:fea:1:2d0:58ff:fe87:8e02` |
| EUROPA `G0/1` | `2001:b0ba:fea:2:260:3eff:fe2c:b702` |

These are the default gateways configured on the DNS and web servers, respectively.

### 3.4 Server configuration

| Setting | DNS server | Web server |
|---|---|---|
| IPv6 address | `2001:b0ba:fea:1::10` | `2001:b0ba:fea:2::10` |
| Prefix length | `64` | `64` |
| Default gateway | `2001:b0ba:fea:1:2d0:58ff:fe87:8e02` | `2001:b0ba:fea:2:260:3eff:fe2c:b702` |
| DNS server field | `2001:b0ba:fea:1::10` | `2001:b0ba:fea:1::10` |

---

## 4. Router configuration concepts

### 4.1 Enable IPv6 routing

On each router:

```text
enable
configure terminal
ipv6 unicast-routing
```

This enables IPv6 packet forwarding between interfaces. Giving an interface an IPv6 address alone does not make the router forward packets between networks.

### 4.2 Configure interfaces

An interface is assigned an IPv6 address and enabled:

```text
interface gigabitEthernet 0/1
ipv6 address 2001:beba:cafe:1::1/64
no shutdown
exit
```

The exact address depends on the router and LAN. The command `no shutdown` enables the interface administratively.

### 4.3 EUI-64

The DNS LAN interface on ANTARTIDA and the web LAN interface on EUROPA use this pattern:

```text
interface gigabitEthernet 0/1
ipv6 address 2001:b0ba:fea:1::/64 eui-64
no shutdown
```

On EUROPA, the prefix is `2001:b0ba:fea:2::/64`.

**Important:** EUI-64 generates the interface identifier from the interface MAC address. It does not change the network prefix. The server can use a manually assigned address such as `::10` in the same `/64`.

### 4.4 DHCPv6 on ASIA and COLOMBIA

The routers have DHCPv6 pools for the PC LANs. The pools define the prefix and DNS/domain information; the LAN interfaces associate the pool with the correct LAN and set the managed-configuration flag.

Example structure:

```text
ipv6 dhcp pool LAN1_POOL
address prefix 2001:beba:cafe:1::/64
dns-server 2001:b0ba:fea:1::10
domain-name mywebsite.edu.co
exit

interface gigabitEthernet 0/1
ipv6 address 2001:beba:cafe:1::1/64
ipv6 nd managed-config-flag
ipv6 dhcp server LAN1_POOL
no shutdown
exit
```

ASIA has pools for LAN 1 and LAN 2. COLOMBIA has a pool for LAN 3. The PCs must be set to DHCP/DHCPv6 in Packet Tracer and their actual address, gateway, and DNS fields should be checked.

### 4.5 Static routing

Each router needs routes to remote networks that are not directly connected. A static route has the general format:

```text
ipv6 route <remote-prefix> <next-hop-address>
```

For example, ANTARTIDA routes ASIA LAN 1 traffic to ASIA's serial address:

```text
ipv6 route 2001:beba:cafe:1::/64 2001:b0ba:fea:10::2
```

The final working route plan is:

| Router | Remote network | Next hop |
|---|---|---|
| ANTARTIDA | ASIA LAN 1 `2001:beba:cafe:1::/64` | `2001:b0ba:fea:10::2` |
| ANTARTIDA | ASIA LAN 2 `2001:beba:cafe:2::/64` | `2001:b0ba:fea:10::2` |
| ANTARTIDA | COLOMBIA LAN `2001:beba:cafe:3::/64` | `2001:b0ba:fea:11::2` |
| ANTARTIDA | Web LAN `2001:b0ba:fea:2::/64` | `2001:b0ba:fea:12::2` |
| ASIA | DNS LAN `2001:b0ba:fea:1::/64` | `2001:b0ba:fea:10::1` |
| ASIA | COLOMBIA LAN `2001:beba:cafe:3::/64` | `2001:b0ba:fea:10::1` |
| ASIA | Web LAN `2001:b0ba:fea:2::/64` | `2001:b0ba:fea:10::1` |
| COLOMBIA | DNS LAN `2001:b0ba:fea:1::/64` | `2001:b0ba:fea:11::1` |
| COLOMBIA | ASIA LAN 1 `2001:beba:cafe:1::/64` | `2001:b0ba:fea:11::1` |
| COLOMBIA | ASIA LAN 2 `2001:beba:cafe:2::/64` | `2001:b0ba:fea:11::1` |
| COLOMBIA | Web LAN `2001:b0ba:fea:2::/64` | `2001:b0ba:fea:11::1` |
| EUROPA | DNS LAN `2001:b0ba:fea:1::/64` | `2001:b0ba:fea:12::1` |
| EUROPA | ASIA LAN 1 `2001:beba:cafe:1::/64` | `2001:b0ba:fea:12::1` |
| EUROPA | ASIA LAN 2 `2001:beba:cafe:2::/64` | `2001:b0ba:fea:12::1` |
| EUROPA | COLOMBIA LAN `2001:beba:cafe:3::/64` | `2001:b0ba:fea:12::1` |

To verify routes:

```text
show ipv6 route
```

- `C` = directly connected network.
- `L` = local address assigned to the router.
- `S` = static route.

A key troubleshooting lesson from this project: a router can have working interfaces but still fail to deliver traffic to a remote LAN if the necessary routes are missing. Return traffic also needs a valid route.

---

## 5. DNS and website configuration

### 5.1 DNS server

On the DNS server, configure its static IPv6 address and gateway as shown in the addressing table. In **Services → DNS**, switch DNS **On** and add the domain record:

| DNS field | Value |
|---|---|
| Name | `www.mywebsite.edu.co` |
| Record type | `AAAA`, if available in the Packet Tracer interface |
| Address | `2001:b0ba:fea:2::10` |

An AAAA record maps a domain name to an IPv6 address. If the Packet Tracer version presents a different DNS record interface, verify the available record types before proceeding.

### 5.2 Web server

On the web server, configure the static IPv6 address, prefix, gateway, and DNS server from the table. Then open **Services → HTTP** and turn HTTP **On**. Edit `index.html` so the page displays a simple message such as:

```html
<html>
  <body>
    <h1>Hello</h1>
    <p>Welcome to my website!</p>
  </body>
</html>
```

Save the page if the editor requires it.

### 5.3 Website test

From a PC, open **Desktop → Web Browser** and visit:

```text
http://www.mywebsite.edu.co
```

The expected result is a page displaying “Hello”. A successful ping to the server IP confirms basic IPv6 reachability, but it does not by itself prove that DNS name resolution or HTTP is configured correctly.

---

## 6. Verification checklist

| Test | How to perform it | Expected result |
|---|---|---|
| Router interface status | `show ipv6 interface brief` | Used interfaces show `up/up` and assigned addresses |
| ASIA gateway | From an ASIA PC: `ping 2001:beba:cafe:1::1` | Replies |
| COLOMBIA gateway | From a COLOMBIA PC: `ping 2001:beba:cafe:3::1` | Replies |
| DNS server reachability | From a PC: `ping 2001:b0ba:fea:1::10` | Replies |
| Web server reachability | From a PC: `ping 2001:b0ba:fea:2::10` | Replies |
| Routing | On each router: `show ipv6 route` | Connected, local, and needed static routes appear |
| DNS resolution | Browse to `http://www.mywebsite.edu.co` | Domain resolves to the web server |
| HTTP service | Open the website in a PC browser | Page displays “Hello” |
| Save the project | Save the `.pkt` file | File preserves the configuration |

### Current status

The following have been verified in the conversation:

- All used router interfaces were reported as `up/up`.
- ASIA and COLOMBIA PCs could ping their local IPv6 gateways.
- An ASIA PC successfully pinged both `2001:b0ba:fea:1::10` (DNS server) and `2001:b0ba:fea:2::10` (web server) after ANTARTIDA's missing static routes were added.

Still verify DNS record creation, HTTP service/page configuration, and browsing by domain name before calling the entire web application complete.

---

## 7. What to remember for future projects

1. **Plan first:** write down every LAN prefix, gateway, host address, and serial-link address before configuring devices.
2. **Use unique prefixes:** every separate LAN or point-to-point link needs its own subnet.
3. **Enable IPv6 routing:** use `ipv6 unicast-routing` on routers that must forward IPv6 traffic.
4. **Enable interfaces:** assign the correct address and use `no shutdown`.
5. **Route in both directions:** a request may reach a server, but the reply still needs a route back to the originating PC.
6. **Test one layer at a time:** interface status → local gateway → remote router → server IP → DNS → HTTP.
7. **Know the difference:** EUI-64 generates the interface identifier; DHCPv6 helps configure clients; static routes tell routers where to forward remote traffic; DNS maps a name to an IP address; HTTP serves the web page.
8. **Save checkpoints:** save the Packet Tracer file after each major working milestone.

## 8. Deliverables

Before submission, confirm that you have:

- [ ] Saved the Packet Tracer topology as a `.pkt` file.
- [ ] Captured a clear screenshot of the complete topology and device labels.
- [ ] Saved the exact final CLI commands for ANTARTIDA, ASIA, COLOMBIA, and EUROPA as four separate `.txt` files.
- [ ] Configured and tested the DNS AAAA record.
- [ ] Configured and tested the HTTP page.
- [ ] Verified that `http://www.mywebsite.edu.co` opens from a PC.

