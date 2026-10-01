# Room Walkthrough- Tcpdump the Basics
Before you read: there are some spoilers here- try to understand everything before actually typing in the answer!
## Introduction
I started my attacker machine and lab machine first, and connected to ssh. (basically running “ssh@ip_address_of_lab_machine”, then providing the credentials once prompted.
## Task 2- Basic Packet Capture
Then there is Task 2, which provides a brief overview of the basic command line options in tcpdump.
The -n option is really important (as a complete beginner myself) for the following tasks. (Which displays numerical IP addresses only, and apparently speeds up the output and makes it easier to read.)
## Task 3- Filtering Expression
Straight on to Task 3, filtering expressions. Well obviously we get an overview of “filtering expressions”, by host, port and protocol.
For the question, “_How many packets in traffic.pcap use the ICMP protocol?_”, I used

` tcpdump -r traffic.pcap icmp -n | wc -l` 

Explaining:
* tcpdump: this is the computer program
* -r: read
* traffic.pcap: the packet capture file
* icmp: the protocol
* -n: see Task 2
* wc: word count
* -l: count lines only
As supplementary, for -l, if you need it you can also replace it with -w (count words only) or -c (count bytes). For icmp, you can also replace it with other protocols; e.g. tcp, udp, etc.
---
Next question, “_What is the IP address of the host that asked for the MAC address of 192.168.124.137?_”
I used

` tcpdump -r traffic.pcap arp -e -n` 
* “arp” stands for ARP packets, which allows us to focus on the “Address Resolution Protocol traffic”, which is used to map IP addresses to MAC addresses
* “-e”- link-layer (Ethernet) headers in the output. This is important for seeing MAC addresses associated with the ARP requests and responses.
And so seeing the output was pretty self-explanatory.
---
Third question: _What hostname (subdomain) appears in the first DNS query?_
My command was

`tcpdump -r traffic.pcap port 53 -n -c 1`
* Port 53 is the default port used for DNS (Domain Name System) queries and responses.
* -c specifies the count of packets to capture. Basically, it tells tcpdump to stop reading packets after capturing the specified number.
* 1 is the value provided to -c, telling tcpdump to only capture the first packet that matches the criteria.
## Task 4- Advanced Filtering
Some advanced filtering, as you can see; but once you understand how to filter based on the contents of a header byte, it’s pretty simple.
_How many packets have only the TCP Reset (RST) flag set?_

`tcpdump -r traffic.pcap 'tcp[tcpflags] == tcp-rst' -n | wc -l`
* ‘tcp[tcpflags] == tcp-rst’ is the part we want to focus on, capturing TCP packets with only the RST (reset) flag set, while all the other flags are unset.
---
_What is the IP address of the host that sent packets larger than 15000 bytes?_
I wrote this-

`tcpdump -r traffic.pcap 'greater 15000' -n | awk '{print $3}' | sort | uniq`
* awk ‘{print $3}’ : extracts the third column from the output, which typically contains the IP address (we haven’t learned this yet in the room, but this is easier so if you were wondering, it processes text files and allows you to extract specific fields from output easily.)
* sort: makes identical IP addresses adjacent
* uniq: to remove duplicate lines. (It only removes adjacent duplicates, so it is typically used after sort.)
We haven’t covered these by task 4, so this is if you don’t understand:

`tcpdump -r traffic.pcap 'greater 15000' -n`
And you can scroll and read manually to get the answer.

---
_What is the MAC address of the host that sent an ARP request?_
I executed this

`tcpdump -r traffic.pcap arp -e -n`
* arp: this filters the output to show only ARP packets, which are used for mapping IP addresses to MAC addresses
* e: this option includes the link-layer (MAC) header in the output
## Conclusion
We officially made it through the TCPdump basics! Although we only covered the basics, this room really uncovers the mass extent of Tcpdump. 
I would recommend doing also the Wireshark room to learn a bit more…
If you found this walkthrough helpful, feel free to share it or leave a comment! See you in the next post!
P.S. pls star this repository if you found it helpful!


