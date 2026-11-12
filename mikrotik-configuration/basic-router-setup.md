# MikroTik RouterOS – Basic Router Configuration

## 1. Introduction

This guide demonstrates the basic configuration of a MikroTik router using RouterOS v7.

It covers initial router setup, LAN configuration, WAN connectivity, DHCP services, DNS, NAT, and basic firewall rules.

The purpose is to provide a practical reference for learning and documenting MikroTik network administration.

## 2. Network Topology

The example uses the following network design:

```text
             INTERNET
                 |
                 |
          [ WAN - ether1 ]
           MikroTik Router
          [ LAN - ether2-5 ]
                 |
           LAN Network
          192.168.10.0/24
                 |
          Client Devices
```

### IP Addressing Plan

| Device / Interface | IP Address |
|---|---|
| Router LAN | 192.168.10.1/24 |
| Network Address | 192.168.10.0/24 |
| DHCP Pool Start | 192.168.10.100 |
| DHCP Pool End | 192.168.10.200 |
| WAN Interface | ether1 |
| LAN Interfaces | ether2-ether5 |

## 3. Initial Router Configuration

Connect to the MikroTik router using WinBox or its terminal.

Set the router identity:

```routeros
/system identity set name=Office-Router
```

## 4. Configure LAN Bridge

Create a bridge:

```routeros
/interface bridge add name=bridge-LAN
```

Add LAN interfaces:

```routeros
/interface bridge port
add bridge=bridge-LAN interface=ether2
add bridge=bridge-LAN interface=ether3
add bridge=bridge-LAN interface=ether4
add bridge=bridge-LAN interface=ether5
```

## 5. Configure LAN IP Address

Assign an IP address to the LAN bridge:

```routeros
/ip address add address=192.168.10.1/24 interface=bridge-LAN
```

This address will serve as the default gateway for LAN devices.

## 6. Configure DHCP Server

Create a DHCP address pool:

```routeros
/ip pool add name=LAN-POOL ranges=192.168.10.100-192.168.10.200
```

Configure the DHCP network:

```routeros
/ip dhcp-server network add address=192.168.10.0/24 gateway=192.168.10.1 dns-server=192.168.10.1
```

Create the DHCP server:

```routeros
/ip dhcp-server add name=LAN-DHCP interface=bridge-LAN address-pool=LAN-POOL disabled=no
```

Now connected LAN devices can receive IP addresses automatically.

## 7. Configure WAN Connectivity

For an internet connection that provides an IP address through DHCP:

```routeros
/ip dhcp-client add interface=ether1 disabled=no
```

The router should receive its WAN IP address and default route from the upstream network.

## 8. Configure DNS

Set public DNS resolvers:

```routeros
/ip dns set servers=1.1.1.1,8.8.8.8 allow-remote-requests=yes
```

This allows LAN devices to use the MikroTik router for DNS resolution.

## 9. Configure NAT (Masquerade)

Enable LAN internet access through the WAN interface:

```routeros
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="LAN Internet NAT"
```

Masquerading translates private LAN addresses into the router's WAN address.

## 10. Configure Basic Firewall Rules

First, allow established and related connections:

```routeros
/ip firewall filter add chain=input connection-state=established,related action=accept comment="Allow Established and Related"
```

Drop invalid connections:

```routeros
/ip firewall filter add chain=input connection-state=invalid action=drop comment="Drop Invalid Input"
```

Allow LAN access to the router:

```routeros
/ip firewall filter add chain=input in-interface=bridge-LAN action=accept comment="Allow LAN Access"
```

Drop other input traffic:

```routeros
/ip firewall filter add chain=input action=drop comment="Drop Other Input Traffic"
```

Allow established and related forwarded traffic:

```routeros
/ip firewall filter add chain=forward connection-state=established,related action=accept comment="Allow Established Forward"
```

Drop invalid forwarded traffic:

```routeros
/ip firewall filter add chain=forward connection-state=invalid action=drop comment="Drop Invalid Forward"
```

Drop new connections from WAN that are not destination-NATed:

```routeros
/ip firewall filter add chain=forward in-interface=ether1 connection-state=new connection-nat-state=!dstnat action=drop comment="Block Unsolicited WAN Forward"
```

## 11. Verify the Configuration

Use the following commands to inspect your configuration:

```routeros
/interface print
/ip address print
/ip dhcp-client print
/ip dhcp-server print
/ip route print
/ip dns print
/ip firewall filter print
/ip firewall nat print
```

Test internet connectivity:

```routeros
/ping 1.1.1.1
```

Test DNS resolution:

```routeros
/ping google.com
```

## 12. Expected Results

After successful configuration:

- LAN devices should receive IP addresses automatically.
- LAN devices should use 192.168.10.1 as their gateway.
- The router should obtain a WAN address through DHCP.
- LAN devices should be able to access the internet.
- Basic firewall protection should be active.

## 13. Important Notes

- These commands are intended for a lab router or a clean configuration.
- Confirm your actual interface names before applying commands.
- Do not blindly apply this configuration to a router with an existing setup.
- Existing default firewall rules may conflict with these example rules.
- The example assumes DHCP-based WAN connectivity.
- If your ISP uses PPPoE or static addressing, WAN configuration must be adjusted.
- Apply firewall changes carefully to avoid losing remote management access.
- Use Safe Mode in WinBox or the RouterOS terminal when making remote configuration changes.
- Back up your existing configuration before making changes.

## 14. Conclusion

This guide demonstrates the essential steps involved in configuring a MikroTik router for a basic office LAN environment.

It provides a foundation for more advanced networking topics, including static routing, VLANs, firewall management, VPNs, bandwidth control, and network monitoring.

---

**Author:** MD Sakib Hosen

**Professional Portfolio:** https://md-sakib-hosen.github.io/

**GitHub:** https://github.com/Md-Sakib-Hosen



Important technical note

This is a learning configuration, not a universal production-ready firewall. Test it on a clean lab router, confirm your interfaces and existing firewall rules, and take a backup before applying it.
