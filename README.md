# OpenMPTCPDocs

# Home Internet Aggregation Architecture (OpenMPTCProuter + pfSense + Proxmox)

![Architecture Diagram](./A_2D_digital_diagram_showcases_a_home_internet_agg.png)

## Overview

This setup is designed for load balancing and multipath TCP (MPTCP) aggregation using:

- **OpenMPTCProuter (OMR)** for aggregating multiple WANs and routing through a remote VPS
- **pfSense** as the firewall, gateway, and LAN controller
- **Proxmox VE** hosting pfSense and other infrastructure VMs
- Three uplink sources: **Starlink**, **T-Mobile**, and **Midco**
- **VPS (VaderNode)** acting as the MPTCP tunnel endpoint

---

## Architecture Summary

### WAN Interfaces (OMR)
- **eth1** → Starlink (192.168.1.1)
- **eth2** → T-Mobile (192.168.12.1)
- **eth3** → Midco (static or DHCP; e.g. 192.168.100.1)

### OpenMPTCProuter (OMR)
- Aggregates Starlink, T-Mobile, and Midco using Multipath TCP
- Routes all WAN uplinks through a tunnel to VPS on port `65500`
- Admin LAN IP: `192.168.40.60`
- LAN subnet: `192.168.40.0/24`

### VPS (VaderNode)
- Public IP: `129.x.x.x`
- 2 Gbps symmetric bandwidth
- Runs MPTCP-enabled Linux kernel and `omr-server`
- Handles incoming bonded traffic from OMR

### pfSense (Proxmox VM)
- WAN/OPT interface connected to OMR LAN (`192.168.40.60`)
- Manages local LAN (`192.168.1.0/24`) and other test subnets (e.g. `192.168.42.0/24`)
- Can be configured for:
  - Load balancing between WANs
  - Gateway monitoring
  - Static route injection or policy-based routing
  - NAT and firewalling for internal VMs

### Proxmox
- Hosts pfSense VM, monitoring tools, and various Linux service VMs
- Mellanox NIC handles upstream traffic to pfSense and internet
- OMR and pfSense interfaces bridged to physical and virtual adapters as needed

---

## Known Issues

- Some services (e.g. Microsoft.com) may block traffic from VPS (data center IP ranges)
- Without `mptcpize`, many applications will default to a single subflow
- Proxmox virtio interfaces can reduce throughput if not tuned properly

---

## Future Enhancements

- Add a dedicated test LAN on pfSense
- Use VLANs to isolate WAN sources within Proxmox
- Explore mptcpize and BPF-based schedulers for better MPTCP performance
- Investigate using Hurricane Electric IPv6 tunnels to route around data center restrictions
- Deploy a home MPTCP termination VPS to avoid third-party hosting
