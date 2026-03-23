---
description: 'Category: Forensics (Easy)'
---

# Ping2Win

## Description

```
Pong!
```

## Walkthrough

We are provided with a network capture file, which we can open in Wireshark.

<figure><img src="../../.gitbook/assets/image (371).png" alt=""><figcaption></figcaption></figure>

Immediately we can see a huge amount of ICMP packets, which looks interesting. If we click on the first ICMP packet, we don't see anything weird or out-of-the-ordinary. However, if we start scrolling down, we start seeing some unusual patterns in the packet's payload field.

<figure><img src="../../.gitbook/assets/image (360).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (361).png" alt=""><figcaption></figcaption></figure>

Each packet contains 2 bytes located at the same location, which could suggest that ICMP tunneling is happening in the capture. If we scroll down further, we can see the following packet.

<figure><img src="../../.gitbook/assets/image (374).png" alt=""><figcaption></figcaption></figure>

The equals sign there suggests that the payload could be encoded in Base64, then fragmented in chunks of 2 characters then sent via ICMP. We can extract the bytes and decode it using this Python script

{% code overflow="wrap" %}
```python
#!/usr/bin/env python3
from scapy.all import rdpcap, ICMP
import base64
import sys

def extract_icmp_base64(pcap_file, output_file):
    packets = rdpcap(pcap_file)
    chunks = {}
    for pkt in packets:
        if ICMP in pkt and pkt[ICMP].type == 8:  # Echo Request
            seq = pkt[ICMP].seq
            payload = bytes(pkt[ICMP].payload)
            if payload:
                chunks[seq] = payload
                
    if not chunks:
        print("[!] No ICMP payloads found.")
        return
    # Reassemble in sequence order
    ordered_data = b''.join(chunks[i] for i in sorted(chunks.keys()))
    print(f"[+] Reassembled {len(ordered_data)} bytes of base64 data")
    try:
        decoded = base64.b64decode(ordered_data)
    except Exception as e:
        print(f"[!] Base64 decoding failed: {e}")
        return

    with open(output_file, "wb") as f:
        f.write(decoded)
        
    print(f"[+] Decoded output written to {output_file}")

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} <input.pcap> <output_file>")
        sys.exit(1)
    extract_icmp_base64(sys.argv[1], sys.argv[2])
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (369).png" alt=""><figcaption></figcaption></figure>

There is the flag, plus a message for the participants. Lets submit our flag

<figure><img src="../../.gitbook/assets/image (370).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

This is a very simple packet analysis challenge, inspired from a similar challenge that I solved from CSCV25 (you can see the challenge [here](https://blog.northland.dev/writeups/2025/cscv/coverts)). I added a welcome message to thank the players who solved this challenge manually, without the use of AI. If you're one of these players and reading this, thank you :)
