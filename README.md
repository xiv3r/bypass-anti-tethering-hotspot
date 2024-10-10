 <h1 align="center"><summary>
  
# [Bypass WiFi Anti-Tethering (TTL /HL=1)](https://github.com/xiv3r/anti-tethering-bypasser)

</summary></h1>

<h4 align="center">
 
   Wireless Sharing Protection (anti-tethering) is a mechanism employed by wisp carriers to restrict tethering or hotspot sharing on wifi users by modifying the interface broadcast ttl value to 1 hops. To circumvent these restrictions, iptables can be utilized to modify network traffic characteristics, allowing users to bypass wireless sharing protection (anti-tethering) effectively.

<h1 align="center">
 
 10.0.0.1 ttl=1

👇

Openwrt/Linux WiFi Repeater/Extender mode

👇

10.0.0.1 ttl=64
 </h1>

 <h1 align="center"> Using IPTABLES & IP6TABLES </h1>
 

# Auto Install for Linux
   
    sudo apt update ; sudo apt install curl ; curl https://raw.githubusercontent.com/xiv3r/bypass-anti-tethering/refs/heads/main/install.sh | sudo sh

# Auto Install for OpenWRT

    opkg update ; opkg install curl ; curl https://raw.githubusercontent.com/xiv3r/bypass-anti-tethering/refs/heads/main/install.sh | sh

    
## Note!
- Connect your Router/PC to Internet for Installation.
- Configure your router or pc to Extender/Repeater Mode and done!.
- Openwrt iptables `NAT`  doesn't work properly on version 1.8.7.
- Applicable only for openwrt router, linux and rooted phones.
- Take note that the `wlan0` is your `ISP` and the destination is `eth0`.

# IPTables and IP6Tables to Bypass Anti-Tethering Restriction

```
# IPTABLES for IPv4 WISP with Anti-Tethering

# Change incoming TTL=1 to TTL=65 on wlan0
iptables -t mangle -A PREROUTING -i wlan0 -j TTL --ttl-set 65
iptables -t mangle -A POSTROUTING -o wlan0 -j TTL --ttl-set 64

# Enable NAT (Masquerade) for eth0
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Allow forwarding between wlan0 and eth0
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT

#__________________________________________________________________

# IP6TABLES for IPv6 WISP with Anti-Tethering

# Change incoming hop limit=1 to hop limit=65 on wlan0
ip6tables -t mangle -A PREROUTING -i wlan0 -j HL --hl-set 65
ip6tables -t mangle -A POSTROUTING -o wlan0 -j HL --hl-set 64

# Allow forwarding between wlan0 and eth0
ip6tables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
ip6tables -A FORWARD -i eth0 -o wlan0 -j ACCEPT

```

# How to check?
• IPv4 iptables
    
    iptables -vnL --line-numbers

• IPv6 ip6tables
   
    ip6tables -vnL ---line-numbers

# Features
- Bypass ISP Hotspot sharing restriction
- Support 65 Hops Nodes
- Can Shared or tethered across multiple devices
- Free Internet
    
# Tested on
- Termux Custom Bridge interfaces
- All OpenWRT/XWRT/IMWRT
- All Debian based Distros
- All Arch based Distro
- Almost all OS supported by Iptables

# How to clear Iptables existing rules?
• IPv4 iptables
    
    iptables -F
    iptables -t mangle -F
    
• IPv6 ip6tables
   
    ip6tables -F
    ip6tables -t mangle -F

<h1 align="center"> Using NFTABLES </h1>

To achieve the setup where incoming packets with TTL=1 on the wlan0 interface are modified to have TTL=64 and forwarded to the eth0 interface, and the outgoing packets are modified with TTL=64 when sent back from eth0 to wlan0, you can configure nftables as follows:

1. Install nftables (if not installed)

       opkg update ; opkg install nftables kmod-nft-nat kmod-nft-core kmod-nft-nat kmod-nfnetlink

3. Configure the nftables Rules
Here is a basic nftables configuration to change TTL and allow forwarding between wlan0 and eth

```bash
nft add table inet custom_table

# Prerouting: Change TTL on incoming packets from wlan0
nft add chain inet custom_table prerouting { type filter hook prerouting priority 0 \; }
nft add rule inet custom_table prerouting iif "wlan0" ip ttl set 64

# Postrouting: Enable masquerading on eth0 and set outgoing TTL for wlan0
nft add chain inet custom_table postrouting { type nat hook postrouting priority 100 \; }
nft add rule inet custom_table postrouting oif "eth0" masquerade
nft add rule inet custom_table postrouting oif "wlan0" ip ttl set 64

# Forwarding: Allow traffic between wlan0 and eth0 in both directions
nft add chain inet custom_table forward { type filter hook forward priority 0 \; }
nft add rule inet custom_table forward iif "wlan0" oif "eth0" accept
nft add rule inet custom_table forward iif "eth0" oif "wlan0" accept
```

4. Create the nftables rules file
Create or edit the nftables configuration file

       vi /etc/nftables.conf

6. Ensure nftables Service is Enabled
To ensure nftables starts on boot and the rules persist across reboots, enable and start the nftables service:

       chmod +x /etc/nftables.conf
   
Explanation:
Prerouting chain: Incoming packets on wlan0 with TTL=1 are changed to TTL=64 before forwarding.
Postrouting chain: Outgoing packets through wlan0 are set to TTL=64.
Forward chain: Allows forwarding between wlan0 and eth0 in both directions.
    
