# OpenMPTCProuter + pfSense + Proxmox: Implementation Notes

## Objective

To implement a robust multipath internet aggregation solution at home using:
- **OpenMPTCProuter (OMR)** to bond Starlink, T-Mobile, and Midco uplinks
- **pfSense** to serve as the firewall/router and testbed for future subnets
- **Proxmox** as the host hypervisor for virtualized infrastructure
- A remote **VPS** (VaderNode) as the MPTCP tunnel endpoint

---

## Key Components

| Component        | Role                                              |
|------------------|---------------------------------------------------|
| OpenMPTCProuter  | Aggregates WANs, tunnels via MPTCP to the VPS     |
| pfSense (VM)     | LAN controller, firewall, future subnet testing   |
| Proxmox          | Hypervisor, bridges interfaces, hosts VMs         |
| VPS (VaderNode)  | Remote endpoint for MPTCP traffic                 |
| WAN Sources      | Starlink, T-Mobile, Midco                         |

---

## Setup Summary

1. **OMR flashed on embedded hardware/router**
   - Connected `eth1` (Starlink), `eth2` (T-Mobile), `eth3` (Midco)
   - LAN IP set to `192.168.40.60`
   - Tracker configured to point to VPS `129.x.x.x:65500`
   - WAN ports opened: `65000–65550` on VPS for inbound

2. **VPS (VaderNode) configuration**
   - MPTCP kernel verified
   - omr-server running, listening on port `65500`
   - Firewall rules updated to allow the port range
   - Verified MPTCP using `curl https://check.mptcp.dev` and iperf

3. **pfSense setup in Proxmox**
   - Interface `OPT1` tied to OMR LAN
   - Assigned `192.168.40.2`, gateway `192.168.40.60`
   - NAT and DNS resolver settings updated for traffic flow

4. **Testing**
   - iperf confirmed aggregation was functioning
   - Speed tests showed improvement, but bottlenecks found
   - Tuned interface bridging and disabled OpenVPN fallback

---

## Problems Faced & Solutions

### ❌ _Problem:_ MPTCP not functioning, falling back to VPN  
**Cause:** OMR fell back to OpenVPN tunnel when server wasn't reachable.  
**Solution:**  
- Opened full range of ports on VPS (`65000–65550`)
- Restarted the OMR tracker service after configuration changes
- Verified connection state in `omr-admin` logs

---

### ❌ _Problem:_ MPTCP connection marked as Direct Output, not Server  
**Cause:** Server endpoint not reachable, fallback route used  
**Solution:**  
- Ensured server firewall was open
- Restarted OMR tracker
- Confirmed omr-server was listening and actively logging inbound connections

---

### ❌ _Problem:_ MPTCP traffic still not bonding effectively  
**Cause:** Many apps are not MPTCP-aware or OMR uses proxying over VPN  
**Solution:**  
- Verified with `iperf3` (direct to port `65400`)
- Disabled OpenVPN in OMR
- Saw improved, but not full, aggregation speeds

---

### ❌ _Problem:_ VPS download speeds poor despite 2 Gbps line  
**Cause:** OMR server uses proxy/VPN behavior, not native kernel routing  
**Solution:**  
- Bypassed VPN fallback manually
- Considered `mptcpize` to wrap specific commands but avoided for now

---

### ❌ _Problem:_ pfSense couldn’t resolve packages, DNS failure  
**Cause:** Missing gateway or incorrect routing  
**Solution:**  
- Added correct gateway to `OPT1` interface (OMR LAN: `192.168.40.60`)
- Verified routes with `netstat -rn`

---

### ❌ _Problem:_ Some websites blocked (e.g., Microsoft.com)  
**Cause:** VPS IPs are in data center ASNs  
**Solution (Partial):**  
- Consider Cloudflare routing or HE.net IPv6 tunnels in future
- Exploring low-cost residential proxies or hybrid setups

---

## Future Enhancements

- 🧪 Create VLAN test subnets from pfSense for traffic profiling
- 🧩 Use `mptcpize` or advanced schedulers (e.g., BPF-based) for efficiency
- 🌐 Host your own home-based VPS node (mini server) to maintain residential IPs
- 🛠️ Try bonding via WireGuard + Multipath TCP over custom script routing
- ☁️ Explore Cloudflare Tunnels or HE IPv6 to reroute problematic access

---

## Conclusion

This setup effectively aggregates home WANs through a remote VPS, using OMR to provide bandwidth bonding and failover. While performance varies due to how MPTCP is implemented and endpoint behavior, this configuration forms a solid baseline for more advanced, hybrid networking experimentation.
