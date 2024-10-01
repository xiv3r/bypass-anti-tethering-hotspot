 <h1 align="center"> <summary>
      
### [Bypass Anti-Tethering(TTL /HL)](https://github.com/xiv3r/anti-tethering-bypasser)
   
10.0.0.1 ttl=1 => WiFi Repeater/Extender => 10.0.0.1 ttl=64
</summary> </h1>

## Dependencies 

    sudo apt update ; sudo apt install iptables -y

* Note: Iptables `NAT` works properly on version 1.8.10

## Run permanently after Boot `nano /etc/rc.local`
```
# Flush table rules
iptables -F
iptables -t nat -F
iptables -t mangle -F

# Apply TTL 64 for outbound traffic (leaving interface wlan0)
iptables -t mangle -A POSTROUTING -o wlan0 -j TTL --ttl-set 64

# Apply TTL 64 for inbound traffic (entering interface wlan0)
iptables -t mangle -A PREROUTING -i wlan0 -j TTL --ttl-set 64

# Allow forwarding of traffic from wlan0 to eth0
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

# Allow returning traffic from eth0 to wlan0
iptables -A FORWARD -i eth0 -o wlan0 -m state --state ESTABLISHED,RELATED -j ACCEPT

# Optionally, if eth0 is connected to the internet, masquerade outbound traffic on eth0
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

exit 0
```

## How to check?

    iptables -L --line-numbers -v

## How to clear Iptables existing rules?

    iptables -F
    iptables -t nat -F
    iptables -t mangle -F
