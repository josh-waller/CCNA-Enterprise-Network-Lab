Implemented a full two-office enterprise network with VLANs, EtherChannels, routing, services, security, IPv6, and wireless, following the Jeremy’s IT Lab CCNA Mega Lab design.
VLANs and EtherChannels

Built Layer‑2 EtherChannels between distribution pairs in both offices: PAgP (Cisco‑proprietary) in Office A and LACP (open standard) in Office B, with both sides actively negotiating.
Configured all access–distribution links, including EtherChannels, as 802.1Q trunks with DTP disabled, native VLAN 1000, and specific VLANs allowed per office (10, 20, 40, 99 in A; 10, 20, 30, 99 in B).
Set up VTPv2 with JeremysITLab as the domain, using distribution switches as servers and access switches as clients, then created and propagated VLANs for PCs, phones, Wi‑Fi, and management in each office.

Access Layer and WLC Connectivity
Assigned access ports so PCs are in VLAN 10, IP phones in VLAN 20, SRV1 in VLAN 30, and ensured access mode with DTP disabled on all such ports.
Configured ASW‑A1’s link to WLC1 as a trunk supporting Wi‑Fi (VLAN 40) and Management (VLAN 99), with the management VLAN untagged and DTP disabled.
Administratively shut down all unused access and distribution switch ports to improve security and hygiene.

IPv4 Addressing and Layer‑3 Core
Assigned the specified IPv4 addresses to R1 (two Internet DHCP client links, WAN point‑to‑point links, and a Loopback0).
Enabled IPv4 routing on all core and distribution switches and built a Layer‑3 EtherChannel (PortChannel1) between CSW1 and CSW2 using a Cisco‑proprietary protocol, with /30 addressing on the PortChannel.
Configured all given /30 and /32 addresses on CSW1, CSW2, and the four distribution switches, disabling all unused Layer‑3 interfaces.
Manually configured SRV1’s IPv4 settings (10.5.0.4/24, gateway 10.5.0.1) and management SVIs on all access switches in VLAN 99 with the correct default gateways.

HSRP Redundancy
Implemented multiple HSRPv2 groups per office matching each subnet, using the specified VIPs and real router addresses.
Made DSW‑A1 active for Office A management and PC subnets, and DSW‑A2 active for Phones and Wi‑Fi; similarly, DSW‑B1 active for Office B management and PCs, and DSW‑B2 active for Phones and Servers, using increased priority and preemption.

Spanning Tree (Rapid PVST+)
Enabled Rapid PVST+ on all access and distribution switches so each VLAN has its own spanning‑tree instance.
Aligned STP root bridges with HSRP active devices by giving them the lowest STP priority and setting the standby devices one increment higher.
Enabled PortFast and BPDU Guard on all end‑host interfaces, including the WLC1 connection, to speed host convergence and protect against STP attacks.

OSPF and Default Routes
Configured OSPF process 1 in Area 0 on R1’s LAN‑facing interfaces and on all core and distribution switches’ Layer‑3 interfaces.
Set router IDs to loopback addresses, enabled OSPF on all loopbacks (as passive), and made SVIs (except management) passive on distribution switches.
Configured all physical OSPF links (except the Layer‑3 PortChannel) with a non‑broadcast type that avoids DR/BDR election.
Added two recursive IPv4 default routes on R1 toward the Internet, making the G0/1/0 path a floating static by raising its AD, and redistributed the default into OSPF so R1 acts as an ASBR.

DHCP, DNS, NTP, SNMP, Syslog, FTP, SSH, NAT
Built multiple DHCP pools on R1 for each management, PC, phone, and Wi‑Fi subnet in both offices, excluding the first 10 usable addresses and using SRV1 as DNS, with WLC options where required.
Configured the distribution switches as DHCP relays, forwarding client broadcasts to R1’s Loopback0.
On SRV1, created DNS A records for google.com, youtube.com, jeremysitlab.com, and a CNAME for www.jeremysitlab.com pointing to jeremysitlab.com.
Set the jeremysitlab.com domain name and SRV1 as DNS server on all routers and switches.
Made R1 an authenticated NTP stratum‑5 server synchronized to 216.239.35.0, and configured all core, distribution, and access switches to use R1’s loopback as NTP server with key 1 and password ccna.
Configured SNMP read‑only community string SNMPSTRING on all routers and switches.
Set Syslog on all routers and switches to log all severities to SRV1 and to local buffers with 8192 bytes.
Used FTP from R1 with default credentials (cisco/cisco) to download a new IOS image from SRV1, rebooted R1 with the new image, and removed the old IOS from flash.
Enabled SSHv2 on all routers and switches with large modulus RSA keys, a local user database, VTY access restricted to SSH only, synchronous logging, and an ACL that permits only Office A PCs to initiate SSH sessions.
Configured static NAT on R1 mapping SRV1’s inside address to 203.0.113.113 for inbound Internet access.
Built pool‑based dynamic PAT with ACL 2 for the Office A/B PC and phone subnets plus Wi‑Fi, using POOL1 (203.0.113.200–203.0.113.207/29) as the global range, then verified Internet reachability and failover by adjusting OSPF default‑information originate and toggling G0/0/0.

LLDP and CDP
Disabled CDP on all devices and enabled LLDP instead for vendor‑neutral neighbor discovery.
Disabled LLDP transmit on each access switch’s F0/1 access port to limit host visibility.

ACLs and Layer‑2 Security
Created extended ACL OfficeA_to_OfficeB to allow ICMP from Office A PCs to Office B PCs, block all other traffic between those two subnets, and permit all other traffic, applied following best practice for extended ACL placement.
Implemented Port Security on access ports F0/1 with minimal allowed MAC addresses (one for SRV1), a restrict‑style violation mode that drops offending traffic but keeps the port up, notification on violations, and sticky MAC learning saved to the running‑config.
Enabled DHCP Snooping on all access switches for all active VLANs, trusted only appropriate uplink ports, disabled Option 82 insertion, and applied rate limits of 15 pps on untrusted ports and 100 pps on ASW‑A1’s WLC link.
Configured Dynamic ARP Inspection (DAI) on all access switches for active VLANs, trusting the correct ports and enabling all optional validation checks to protect against ARP spoofing.

IPv6 Preparation
Enabled IPv6 routing on R1, CSW1, and CSW2 and assigned IPv6 addresses: 2001:db8:a::2/64 and 2001:db8:b::2/64 on R1’s Internet‑facing links.
Used 2001:db8:a1::/64 and 2001:db8:a2::/64 with EUI‑64 to generate interface IDs on R1–CSW1 and R1–CSW2 internal links, and enabled IPv6 on CSW1/CSW2 PortChannel1 interfaces without assigning explicit IPv6 addresses.
Added two IPv6 default static routes on R1: a recursive route via 2001:db8:a::1 and a higher‑AD fully specified route via 2001:db8:b::1 as a floating backup.
Wireless Configuration
Logged into WLC1’s HTTPS GUI at 10.0.0.7 from a PC using admin/adminPW12.
Created a dynamic interface “Wi‑Fi” on WLC1 for VLAN 40 with an IP of .4 in 10.6.0.0/24, gateway .1, DHCP server 10.0.0.76, and port 1, correcting the address from the video’s duplicate example.
Configured and enabled WLAN ID 1 with profile name and SSID “Wi‑Fi”, using WPA2 with AES and PSK cisco123.
Verified that both LWAPs successfully joined WLC1, acknowledging Packet Tracer’s limitation that wireless clients cannot obtain addresses from the Wi‑Fi DHCP pool.
