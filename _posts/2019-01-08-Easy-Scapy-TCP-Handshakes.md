---
published: true
comments: true
---
Sometimes you may want to manually establish TCP 3-way handshakes when you're using Python Scapy, but wouldn't it be nice to use sockets to maintain the TCP handshake and pass the data to/from Scapy? If you don't really care about the TCP handshake and want that taken care of while you fuzz the data, here's an easy way using a StreamSocket with Scapy:

```python
from scapy.all import *
mysocket = socket.socket()
mysocket.connect(("192.168.18.40",9999))
mystream=StreamSocket(mysocket)
scapypacket=IP(dst="192.168.18.40")/TCP(dport=9000)/fuzz(Raw())
mystream.send(scapypacket)
```
