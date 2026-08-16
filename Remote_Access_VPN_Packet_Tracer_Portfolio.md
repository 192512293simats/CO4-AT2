# Remote-Access VPN Implementation in Cisco Packet Tracer
### Portfolio: Secure Remote Communication via Cisco Easy VPN (IPsec + Xauth)

---

## 0. VPN Mechanism Used — and Why It's the Correct Choice

Packet Tracer does **not** support full Cisco AnyConnect SSL VPN. The only remote-access VPN mechanism it actually supports is:

> **Cisco Easy VPN** — an IPsec-based remote-access VPN where the router acts as an **Easy VPN Server**, and the remote PC's built-in **VPN Client application** (Desktop → VPN Configuration) acts as the **Easy VPN Remote/software client**.

This is real, documented, working functionality in Packet Tracer (used in Cisco's own CCNA Security course labs). It supports:
- IKE Phase 1 (ISAKMP) with pre-shared key group authentication
- IPsec Phase 2 (transform sets, dynamic crypto maps)
- **Xauth** (extended authentication) — username/password check per remote user
- Mode-config — dynamic IP assignment to the remote client from an internal pool
- Reverse-route injection

This matches your required flow exactly:

```
REMOTE USER → INTERNET/WAN → VPN GATEWAY → INTERNAL LAN → INTERNAL RESOURCES
```

No branch routers, no Tunnel1/Tunnel2, no site-to-site crypto maps between two LANs.

---

## 1. Topology Diagram (build this exactly)

```
[Remote Employee PC]
   192.168.100.10/24
         |
         | (Untrusted Remote Network)
         |
   [ISP / WAN Router]  <-- simulates the public Internet
   Fa0/0: 192.168.100.1/24 (faces remote user)
   Fa0/1: 203.0.113.1/30   (public WAN link)
         |
         | (WAN / "Internet" link — untrusted)
         |
   [Remote Access VPN Gateway]  <-- controlled entry point
   Fa0/0: 203.0.113.2/30   (public/WAN side)
   Fa0/1: 192.168.10.1/24  (trusted LAN side)
         |
         |
   [Internal LAN Switch]
         |
   -------------------------------
   |            |                |
[Internal PC] [File/Web Server] [Database Server]
192.168.10.10  192.168.10.20    192.168.10.30
```

**Visual zoning (use Packet Tracer's colored background/box shapes or notes to mark this clearly):**
- 🔴 **Untrusted zone** — Remote Employee PC + ISP Router + WAN link (203.0.113.0/30)
- 🟡 **Security boundary** — Remote Access VPN Gateway (this is the only device with one leg in untrusted space and one leg in trusted space)
- 🟢 **Trusted zone** — Internal LAN Switch + Internal PC + File/Web Server + Database Server

---

## 2. Device List

| # | Device | Packet Tracer Model | Label |
|---|--------|---------------------|-------|
| 1 | Remote Employee PC | PC-PT | **Remote Employee** |
| 2 | WAN simulation router | Router (e.g. 1941) | **ISP / Internet Router** |
| 3 | VPN gateway router | Router (e.g. 1941, needs `securityk9` license for crypto) | **Remote Access VPN Gateway** |
| 4 | Internal switch | Switch (2960) | **Internal LAN Switch** |
| 5 | Internal PC | PC-PT | **Internal Employee PC** |
| 6 | File/Web server | Server-PT | **File/Web Server** |
| 7 | Database server | Server-PT | **Database Server** |

> ⚠️ Important PT-specific note: on the **VPN Gateway router**, run `show version` — if crypto commands are rejected, go to the router's Physical tab → power off → drag in the **security technology package (securityk9)** license module (or choose a router model in PT that ships with crypto support, e.g. 1941 or 2911). This is required for `crypto` commands to appear at all.

---

## 3. Cable Connections

| From | Port | To | Port | Cable Type |
|------|------|----|------|-----------|
| Remote Employee PC | FastEthernet0 | ISP Router | Fa0/0 | Copper Straight-Through |
| ISP Router | Fa0/1 | VPN Gateway | Fa0/0 | Copper Straight-Through (or Serial if you prefer a WAN-style link) |
| VPN Gateway | Fa0/1 | Internal LAN Switch | Fa0/1 | Copper Straight-Through |
| Internal LAN Switch | Fa0/2 | Internal Employee PC | FastEthernet0 | Copper Straight-Through |
| Internal LAN Switch | Fa0/3 | File/Web Server | FastEthernet0 | Copper Straight-Through |
| Internal LAN Switch | Fa0/4 | Database Server | FastEthernet0 | Copper Straight-Through |

---

## 4. IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|--------------|------------------|
| Remote Employee PC | FastEthernet0 | 192.168.100.10 | 255.255.255.0 | 192.168.100.1 |
| ISP Router | Fa0/0 | 192.168.100.1 | 255.255.255.0 | — |
| ISP Router | Fa0/1 | 203.0.113.1 | 255.255.255.252 | — |
| VPN Gateway | Fa0/0 (WAN) | 203.0.113.2 | 255.255.255.252 | — |
| VPN Gateway | Fa0/1 (LAN) | 192.168.10.1 | 255.255.255.0 | — |
| Internal Employee PC | FastEthernet0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| File/Web Server | FastEthernet0 | 192.168.10.20 | 255.255.255.0 | 192.168.10.1 |
| Database Server | FastEthernet0 | 192.168.10.30 | 255.255.255.0 | 192.168.10.1 |
| **VPN Pool (assigned dynamically to remote client's tunnel)** | — | 192.168.50.10–192.168.50.100 | 255.255.255.0 | (assigned via mode-config) |

Note the remote PC keeps its **physical** address (192.168.100.10) on its real NIC. When the VPN connects, Packet Tracer's VPN client establishes a logical tunnel and the client is also reachable via a **virtual pool address** (192.168.50.x) inside the tunnel — this virtual address is what the internal LAN actually "sees" as the source of tunneled traffic.

---

## 5. Base Router Configuration (do this first, before crypto)

### 5.1 ISP / Internet Router
```
enable
configure terminal
hostname ISP-Router

interface FastEthernet0/0
 ip address 192.168.100.1 255.255.255.0
 no shutdown
exit

interface FastEthernet0/1
 ip address 203.0.113.1 255.255.255.252
 no shutdown
exit

end
write memory
```
This router simulates the Internet — it does **not** get a route to the internal LAN (192.168.10.0/24). It only ever sees encrypted IPsec/ISAKMP traffic destined to 203.0.113.2. This is intentional and proves the LAN is unreachable except through the tunnel.

### 5.2 Remote Access VPN Gateway — interfaces
```
enable
configure terminal
hostname VPN-Gateway

interface FastEthernet0/0
 ip address 203.0.113.2 255.255.255.252
 no shutdown
exit

interface FastEthernet0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

end
```

---

## 6. Routing Configuration

```
! On the VPN Gateway: route to reach the remote network's real subnet
! (needed so ISAKMP/IPsec packets can find their way back to the remote PC)
ip route 192.168.100.0 255.255.255.0 203.0.113.1

! On the ISP Router: nothing extra needed — it is directly connected
! to both 192.168.100.0/24 and 203.0.113.0/30. It has NO route to
! 192.168.10.0/24 — this is deliberate. That subnet must only be
! reachable through the encrypted tunnel, never in the clear.
```

The internal PCs/servers don't need extra routes either — their default gateway is the VPN Gateway itself, and the VPN Gateway owns the pool addresses (192.168.50.0/24) via **reverse-route injection**, configured below.

---

## 7. VPN (Easy VPN Server) Configuration — on the VPN Gateway

### 7.1 Enable AAA (required for Xauth user authentication)
```
configure terminal
aaa new-model
aaa authentication login VPN-AUTHEN local
aaa authorization network VPN-AUTHOR local
```

### 7.2 Create local users (these are the remote employees who may connect)
```
username remoteuser1 password 0 CiscoVPN123
username remoteuser2 password 0 CiscoVPN456
```

### 7.3 ISAKMP (IKE Phase 1) policy
```
crypto isakmp policy 10
 encryption aes 256
 hash sha
 authentication pre-share
 group 2
 lifetime 3600
exit
```

### 7.4 ISAKMP client configuration group (this is what the remote PC's "Group Name" / "Group Key" fields must match)
```
ip local pool VPN-POOL 192.168.50.10 192.168.50.100

crypto isakmp client configuration group VPN-REMOTE-GROUP
 key CorpGroupKey2024
 pool VPN-POOL
 acl 101
exit
```

### 7.5 Split-tunnel ACL (only traffic to the internal LAN goes through the tunnel; defines what the ACL 101 above protects)
```
access-list 101 permit ip 192.168.10.0 0.0.0.255 192.168.50.0 0.0.0.255
```

### 7.6 IPsec transform set (Phase 2 encryption/integrity)
```
crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
exit
```

### 7.7 Dynamic crypto map + reverse-route
```
crypto dynamic-map VPN-DYNMAP 10
 set transform-set VPN-SET
 reverse-route
exit
```

### 7.8 Crypto map tying it all together (Xauth + mode-config + dynamic map)
```
crypto map VPN-CMAP client authentication list VPN-AUTHEN
crypto map VPN-CMAP isakmp authorization list VPN-AUTHOR
crypto map VPN-CMAP client configuration address respond
crypto map VPN-CMAP 10 ipsec-isakmp dynamic VPN-DYNMAP
```

### 7.9 Apply the crypto map to the WAN (public) interface
```
interface FastEthernet0/0
 crypto map VPN-CMAP
exit

end
write memory
```

---

## 8. Authentication Configuration — Summary

There are **two layers** of authentication happening, which you should explicitly call out in your portfolio:

1. **Group authentication (IKE Phase 1)** — the remote PC's VPN client must supply the correct **Group Name** (`VPN-REMOTE-GROUP`) and **Group (pre-shared) Key** (`CorpGroupKey2024`) before ISAKMP will even negotiate. This proves the device "belongs" to your organization's VPN policy.
2. **User authentication (Xauth)** — after Phase 1 succeeds, the router prompts for a **username/password** (checked against the local AAA database — `remoteuser1` / `CiscoVPN123`). This proves *who* the specific person is, not just that they have the group key.

This two-factor structure (something the org issued + something the individual knows) is standard Easy VPN behavior and is exactly what Packet Tracer will actually enforce.

---

## 9. Internal Server Configuration

### File/Web Server (Server-PT)
- Desktop → IP Configuration:
  - IP: `192.168.10.20`
  - Subnet: `255.255.255.0`
  - Gateway: `192.168.10.1`
- Services tab → **HTTP**: turn ON (leave default page, or edit `index.html` to say "Internal File/Web Server — Access Confirmed")

### Database Server (Server-PT)
- Desktop → IP Configuration:
  - IP: `192.168.10.30`
  - Subnet: `255.255.255.0`
  - Gateway: `192.168.10.1`
- Services tab → you can simply leave this reachable via ping/Telnet for testing (PT doesn't simulate real SQL); optionally enable the **Telnet/SSH** or **HTTP** service on it just so you can demonstrate an authenticated internal connection over the tunnel.

### Internal Employee PC
- IP: `192.168.10.10` / `255.255.255.0` / Gateway `192.168.10.1`

---

## 10. Remote-User Configuration (this is the actual "VPN client")

On the **Remote Employee PC**:

1. Desktop → IP Configuration:
   - IP: `192.168.100.10`
   - Subnet: `255.255.255.0`
   - Gateway: `192.168.100.1`
2. Desktop → **VPN** (Packet Tracer's built-in VPN client applet):
   - **Group Name:** `VPN-REMOTE-GROUP`
   - **Group Key:** `CorpGroupKey2024`
   - **Host IP (server):** `203.0.113.2`
   - **Username:** `remoteuser1`
   - **Password:** `CiscoVPN123`
3. Click **Connect**.

If everything above is configured correctly, the client negotiates ISAKMP → Xauth prompt is satisfied automatically from the saved credentials → IPsec SA is built → the applet shows **Connected**, and the PC is assigned a tunnel address from `192.168.50.10–100`.

---

## 11. Verification Commands (run on the VPN Gateway)

```
show crypto isakmp sa
show crypto ipsec sa
show crypto session
show crypto map
show ip route
show run | section crypto
```

**What to look for:**
- `show crypto isakmp sa` → state should read `QM_IDLE` (Phase 1 complete)
- `show crypto ipsec sa` → should show non-zero `#pkts encaps` / `#pkts decaps` once traffic flows, and the peer address `203.0.113.x`/remote's negotiated address
- `show ip route` → should now contain a host route to the assigned pool address (192.168.50.x/32) via reverse-route injection

On the **Remote Employee PC**, the VPN applet itself shows a green "Connected" status with the assigned tunnel IP.

---

## 12. Testing Procedure

1. **Before connecting VPN:** from the Remote Employee PC command prompt, `ping 192.168.10.10` (Internal PC) → **should fail** (this proves the LAN is not reachable without the VPN — critical evidence for your portfolio).
2. Open the VPN applet, enter credentials, click **Connect**. Confirm status = Connected.
3. **After connecting:** `ping 192.168.10.10`, `ping 192.168.10.20`, `ping 192.168.10.30` → **should now succeed**.
4. Open a **web browser** on the Remote Employee PC (Desktop → Web Browser) and browse to `http://192.168.10.20` → the internal web page should load.
5. On the VPN Gateway, run the verification commands from Section 11 while traffic is flowing, to capture the SA counters incrementing.
6. Disconnect the VPN applet, then repeat step 1 to confirm access is revoked again.

---

## 13. Expected Output (what you should actually see)

- Pre-VPN ping to internal LAN: `Request timed out.` (4/4 failed)
- Post-VPN ping to internal LAN: `Reply from 192.168.10.10: bytes=32 time=...` (success)
- `show crypto isakmp sa`: one entry, state `QM_IDLE`, peer = ISP-facing negotiated address
- `show crypto ipsec sa`: `#pkts encaps: n, #pkts decaps: n` both incrementing after each ping
- Browser: internal web server's page renders inside the Remote Employee PC's browser tab

---

## 14. Troubleshooting Steps

| Symptom | Likely Cause | Fix |
|---|---|---|
| VPN applet stuck "Connecting" / fails immediately | Group Name/Key mismatch, or wrong Host IP | Re-check `crypto isakmp client configuration group` values against the PC's VPN tab exactly (case-sensitive) |
| Xauth prompt never satisfied / auth fails | Username/password not in local DB, or AAA lists not applied to crypto map | Verify `username` entries and that `client authentication list` points to the right AAA list name |
| ISAKMP SA never forms | `crypto map` not applied to the correct (WAN) interface, or interface is down | `show run interface Fa0/0` on VPN Gateway; confirm `crypto map VPN-CMAP` is present and `no shutdown` |
| Pings fail even after "Connected" | Reverse-route missing, or internal PCs missing default gateway | Confirm `reverse-route` under the dynamic map; confirm internal devices' gateway = 192.168.10.1 |
| Router won't accept `crypto` commands at all | Router image lacks security/crypto license | Power off router, add the security technology package license in Physical tab, power back on |
| Ping to VPN Gateway's public IP fails from remote PC before VPN | Missing base IP config or `no shutdown` on an interface | Recheck Section 5 interface configs on both routers |

---

## 15. Explanation Summary (for your written portfolio narrative)

- **How the remote user authenticates:** Two-stage — (1) the VPN client presents a shared Group Name/Key to pass IKE Phase 1 group authentication, then (2) Xauth challenges the user for an individual username/password checked against the router's local AAA database.
- **How the VPN tunnel is established:** ISAKMP (IKE Phase 1) negotiates a secure management channel using AES-256/SHA/DH group 2 and the pre-shared group key; after Xauth succeeds, mode-config assigns the client a virtual IP from the pool; IPsec (Phase 2) then builds the actual data-protection SA using the `esp-aes esp-sha-hmac` transform set.
- **How encryption protects communication:** All traffic between the remote PC's assigned tunnel address and the internal LAN is encrypted with AES and integrity-checked with SHA-HMAC inside the ESP payload, so the ISP/Internet router only ever sees opaque encrypted packets, never plaintext internal traffic.
- **How the remote user reaches internal resources:** Once the tunnel is up, the reverse-route-injected host route makes the VPN Gateway advertise reachability to the client's pool address; the internal devices' default gateway routes replies back through the gateway into the tunnel, so pings/HTTP requests flow end-to-end through the encrypted path only.
- **How to verify the VPN:** `show crypto isakmp sa` and `show crypto ipsec sa` on the gateway, plus the green "Connected" state and assigned pool IP on the client's VPN applet.
- **How to prove secure communication is working:** The before/after ping test (Section 12, steps 1 and 3) is your strongest portfolio evidence — LAN unreachable without VPN, reachable only after a successful authenticated tunnel is established, with SA packet counters incrementing to show real encrypted traffic flow.

---

## 16. Screenshot List for Portfolio Submission

| # | Screenshot | What to capture |
|---|---|---|
| 1 | **Complete Topology** | Full Logical workspace showing all 7 devices, cabling, and your three color-coded zones (untrusted/gateway/trusted) |
| 2 | **IP Configuration** | Each device's Desktop → IP Configuration window (or a combined `show ip interface brief` from both routers) |
| 3 | **VPN Gateway Configuration** | `show run | section crypto` output on the VPN Gateway CLI |
| 4 | **Authentication Configuration** | `show run | section aaa` plus `show run | include username` on the VPN Gateway |
| 5 | **VPN Verification** | `show crypto isakmp sa` and `show crypto ipsec sa` output, captured *after* a successful connect |
| 6 | **Remote User Connectivity** | The Remote Employee PC's VPN applet showing "Connected" status with assigned pool IP |
| 7 | **Internal Resource Access** | Successful `ping 192.168.10.20` from the Remote PC, and the browser loading the internal web server |
| 8 | **Final Testing/Results** | Side-by-side: failed ping before VPN connect vs. successful ping after connect (best as two stacked screenshots or a before/after composite) |

---

## 17. Consistency Check

Everything in this document uses only devices/commands present in the topology described in Section 1–4: 2 routers (ISP simulating WAN, VPN Gateway as the controlled boundary), 1 switch, 3 end devices, and the built-in PT VPN client applet. No branch routers, no site-to-site tunnels, no components appear in the explanation that aren't in the build.
