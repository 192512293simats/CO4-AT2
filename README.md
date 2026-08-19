Q6)Develop a portfolio explaining the implementation of remote-access VPN for secure communication

Done By Lokesh.A (192512293)

Remote Access VPN using Cisco Packet Tracer

This project demonstrates a secure remote-access VPN using Cisco Easy VPN, IPsec, and Xauth in Cisco Packet Tracer.

What I Built:

Remote Employee PC connected through an ISP/WAN router

VPN Gateway connected to the internal LAN
Internal PC, File/Web Server, and Database Server

IPsec-based encrypted VPN tunnel

Xauth username/password authentication

VPN address pool for remote users

Connectivity testing before and after VPN connection

How It Works:

Remote User → ISP/WAN → VPN Gateway → Internal LAN → Internal Resources

Before connecting to the VPN, the remote user cannot access the internal resources

After successful authentication, the VPN tunnel is established and the remote user can securely access the internal network.

Technologies Used:

Cisco Packet Tracer | Cisco Easy VPN | IPsec | Xauth | AAA | IKE/ISAKMP

Result:


The VPN was successfully verified using show crypto isakmp sa, show crypto ipsec sa, ping tests, and internal web server access.
