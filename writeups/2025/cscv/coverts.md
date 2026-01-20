---
description: 'Solved by: Ryo Soikutsu'
---

# CovertS

## Description

{% code overflow="wrap" %}
```
Dang, I left my computer unlocked, and my friend said he had exfiltrated something to his machine. Luckily, I was capturing the network traffic at the time. Please help me analyze it and find out what secret he took.
```
{% endcode %}

Category: Forensics

## Walkthrough

Unzipping the challenge archive, we are met with a single, massive network capture file

<figure><img src="../../.gitbook/assets/image (248).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (249).png" alt=""><figcaption></figcaption></figure>

Scrolling through the network traffic, it appears that a majority of the traffic appears to be encrypted, as evident by the TLS and QUIC packets. Initially, I expected decryption of the packets to be part of the challenge. However, we were not provided with a SSLKEYLOGFILE to decrypt traffic. Next up, I decided to take a look at all the IP addresses recorded in the network capture

<figure><img src="../../.gitbook/assets/image (250).png" alt=""><figcaption></figcaption></figure>

Out of the entire list of IP addresses, two addresses in particular stood out to me (192.168.192.1 and 192.168.203.91). These two addresses are part of the 192.168/16 range, which is commonly used as a private network range. This lines up with the challenge description of the friend sending data from the challenge creator's computer to their own. We can filter for traffic that originates between these two machines using the following filter

```
ip.addr == 192.168.192.1 and ip.addr == 192.168.203.91
```

<figure><img src="../../.gitbook/assets/image (251).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (252).png" alt=""><figcaption></figcaption></figure>

The combination of one-way traffic, the unusual destination port (TCP port 3239) and the equal signs being transmitted in the last packet implies that data was being encoded in Base64, and then transmitted over the network. If we can extract the 3rd and 4th byte from the end for each packet, we can reconstruct the data being sent over and decode it. Since copying the data from 600+ packets would take too long, we can write a Bash script that uses tcpdump and grep to extract the bytes for us

```bash
tcpdump -nn -r challenge.pcapng -A src 192.168.203.91 and dst 192.168.192.1 and port 3239 > challenge.tcpdump
cat challenge.tcpdump | grep -oP ".{2}\.\.$" > output 
```

The first command dumps out the ASCII text of all packets that originate from 192.168.203.91 and have a destination of 192.168.192.1:3239. The second command extracts all but the last 4 bytes of each line, and writes them out to another file. Finally, we can import the output file into CyberChef, clean up the data and decode it to get the final output

<figure><img src="../../.gitbook/assets/image (253).png" alt=""><figcaption></figcaption></figure>

We even get a cute welcome message from the challenge creator, which I have copied below

{% code overflow="wrap" %}
```
Hello everyone,
How are you doing? A very warm welcome to CSCV2025!

I'm really glad to see you here and I hope you're ready for an exciting event ahead. This CTF is all about challenging your skills, learning new tricks, and of course - having fun along the way. Consider this little message not as a challenge itself, but simply as my way of saying hello to all of you amazing players.

Take a moment, get comfortable, and enjoy the ride. Whether you're here to compete fiercely, to learn something new, or just to have a good time, I hope CSCV2025 will be an unforgettable experience for you (not this challenge, pls forget this sh*t O_O)

And now, without keeping you waiting any longer...

(someone accidentally sent my chal via email so here is your new flag:)

CSCV2025{my_chal_got_leaked_before_the_contest_bruh_here_is_your_new_flag_b8891c4e147c452b8cc6642f10400452}

^_^ sry for the mess
```
{% endcode %}

## Conclusion

This challenge took me about an hour or so to complete, and was one of the more enjoyable challenges of CSCV 2025. The welcome message is very cute, and I wish to see more challenge creators insert such easter eggs in their challenges in future.
