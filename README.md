Remote Access VPN Implementation in Cisco Packet Tracer
Introduction
This project demonstrates how a remote employee can securely access an organization's internal network using a VPN.
The project is implemented in Cisco Packet Tracer using Cisco Easy VPN with IPsec and Xauth. The Remote Employee PC connects through an untrusted WAN network to the VPN Gateway. After successful authentication, the user can access the internal network securely.
The basic communication flow is:
Remote Employee PC → ISP/WAN → VPN Gateway → Internal LAN → Internal Resources
Network Topology
The network contains a Remote Employee PC, an ISP Router, a VPN Gateway, an Internal Switch, an Internal PC, a File/Web Server and a Database Server.
The Remote Employee PC uses the address 192.168.100.10.
The ISP Router connects the remote network to the WAN using 203.0.113.1.
The VPN Gateway uses 203.0.113.2 on the WAN side and 192.168.10.1 on the internal LAN side.
The internal network contains:
Internal PC – 192.168.10.10
File/Web Server – 192.168.10.20
Database Server – 192.168.10.30
The VPN clients receive an address from the VPN pool 192.168.50.10 to 192.168.50.100. �
Remote_Access_VPN_Packet_Tracer_Portfolio.md
VPN Technology Used
Cisco Easy VPN is used because it provides remote-access VPN functionality in Cisco Packet Tracer.
The VPN uses IPsec to protect the communication and Xauth to authenticate individual users.
The main security mechanisms used are:
IKE Phase 1 for VPN negotiation
Pre-shared key authentication
Xauth username and password authentication
IPsec encryption
Dynamic crypto maps
VPN address pool
Reverse-route injection
IP Configuration
The Remote Employee PC is configured with:
IP Address: 192.168.100.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.100.1
The ISP Router uses:
Fa0/0: 192.168.100.1
Fa0/1: 203.0.113.1
The VPN Gateway uses:
Fa0/0: 203.0.113.2
Fa0/1: 192.168.10.1
The internal devices use the VPN Gateway address 192.168.10.1 as their default gateway.
VPN Gateway Configuration
AAA is enabled on the VPN Gateway for user authentication.
Two VPN users are created:
remoteuser1 with password CiscoVPN123
remoteuser2 with password CiscoVPN456
The IKE policy uses AES-256 encryption, SHA hashing, pre-shared authentication and Diffie-Hellman group 2.
The VPN address pool is configured from 192.168.50.10 to 192.168.50.100.
The VPN group name is VPN-REMOTE-GROUP and the group key is CorpGroupKey2024.
IPsec uses AES encryption with SHA-HMAC for integrity.
The dynamic crypto map is also configured with reverse-route injection so that the VPN Gateway can reach the dynamically assigned VPN client addresses. �
Remote_Access_VPN_Packet_Tracer_Portfolio.md
Authentication
There are two stages of authentication in this VPN.
First, the remote client provides the VPN Group Name and Group Key. This allows the IKE negotiation to begin.
Second, Xauth asks for the individual user's username and password. These credentials are checked against the local AAA database on the VPN Gateway.
This provides an additional layer of security because the user needs both the VPN group credentials and valid individual login credentials. �
Remote_Access_VPN_Packet_Tracer_Portfolio.md
Remote VPN Connection
On the Remote Employee PC, the VPN configuration is entered using the VPN application.
The following details are used:
Host IP: 203.0.113.2
Group Name: VPN-REMOTE-GROUP
Group Key: CorpGroupKey2024
Username: remoteuser1
Password: CiscoVPN123
After entering the correct details, the user connects to the VPN.
Once the connection is successful, the VPN client receives an IP address from the VPN pool.
VPN Verification
The VPN Gateway can be checked using the following commands:
show crypto isakmp sa
show crypto ipsec sa
show crypto session
show crypto map
show ip route
The ISAKMP security association should show the VPN session in the appropriate established state.
The IPsec security association should show packet counters increasing when traffic is sent through the VPN.
This confirms that encrypted traffic is actually passing through the VPN tunnel. �
Remote_Access_VPN_Packet_Tracer_Portfolio.md
Testing
The VPN is tested before and after establishing the connection.
Before connecting the VPN, the Remote Employee PC should not be able to directly access the internal network.
After connecting the VPN, the Remote Employee PC should be able to ping the internal devices.
The following addresses can be tested:
192.168.10.10
192.168.10.20
192.168.10.30
The internal web server can also be accessed using 192.168.10.20.
If the ping succeeds and the internal web page loads, it confirms that the VPN connection is working correctly.
Result
The project successfully demonstrates secure remote access using Cisco Easy VPN.
Before the VPN connection, the remote user cannot directly access the internal network. After authentication and establishment of the IPsec tunnel, the remote user can securely communicate with the internal network.
The project demonstrates authentication, encryption, secure tunneling and controlled access to internal resources.
Conclusion
This project shows how a remote employee can securely connect to an organization's internal network through an untrusted WAN.
Cisco Easy VPN provides the remote-access connection, IPsec protects the communication, and Xauth provides user authentication.
The successful ping and internal web server access after VPN connection confirm that the secure remote-access VPN is working as intended.  - Subnet: `255.255.255.0`
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
