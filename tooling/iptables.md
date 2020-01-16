# IPTABLES

## List rules (incl. NAT and chain #)

```
iptables -L -n -v -t nat --line-numbers
```

## Delete rules

```
iptables -D <chain> <number>
```

E.g.: `iptables -D FORWARD 2`

## Port-Forwarding

```
iptables -A PREROUTING -t nat -i eth0 -p udp --dport 4500 -j DNAT --to 192.168.170.195:4500
iptables -A FORWARD -p udp -d 192.168.170.195 --dport 4500 -j ACCEPT
```

## Enable NAT

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```