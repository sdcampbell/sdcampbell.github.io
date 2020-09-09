---
published: true
comments: true
---
The following Golang code demonstrates how to take a network address string in CIDR format and return a slice of strings containing host addresses. Note that I found this code on the Golang Playground (I don't know specifically who to attribute it to) and edited it slightly and added my own comments so that I was sure to understand what the code was doing before posting it here for my notes.


```golang
package main

import (
	"encoding/binary"
	"fmt"
	"log"
	"net"
)

func main() {
	hosts := cidrHosts("192.168.1.0/24")
    	for _, host := range hosts {
    		fmt.Println(host)
    	}
}

func cidrHosts(netw string) []string {
	// convert string to IPNet struct
	_, ipv4Net, err := net.ParseCIDR(netw)
	if err != nil {
		log.Fatal(err)
	}
	// convert IPNet struct mask and address to uint32
	mask := binary.BigEndian.Uint32(ipv4Net.Mask)
	// find the start IP address
	start := binary.BigEndian.Uint32(ipv4Net.IP)
	// find the final IP address
	finish := (start & mask) | (mask ^ 0xffffffff)
	// make a slice to return host addresses
	var hosts []string
	// loop through addresses as uint32.
    // I used "start + 1" and "finish - 1" to discard the network and broadcast addresses.
	for i := start + 1; i <= finish-1; i++ {
		// convert back to net.IPs
        // Create IP address of type net.IP. IPv4 is 4 bytes, IPv6 is 16 bytes.
		ip := make(net.IP, 4)
		binary.BigEndian.PutUint32(ip, i)
		hosts = append(hosts, ip.String())
	}
	// return a slice of strings containing IP addresses
	return hosts
}
```

Output:

```
192.168.1.1
192.168.1.2
192.168.1.3
192.168.1.4
192.168.1.5
192.168.1.6
192.168.1.7
192.168.1.8
192.168.1.9
192.168.1.10
...
192.168.1.254
```
