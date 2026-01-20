---
description: 'Creator: wrenches'
---

# KILLSWITCH

## Description

```
the user JRJRJR's been up to no good... what're they hiding?
```

Category: Forensics (Easy)

## Walkthrough

We are given a Linux disk image, which we can open in FTK Imager

<figure><img src="../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

Immediately, one of the directories catches our attention, the <mark style="color:red;">**/**</mark><mark style="color:red;">secret</mark> folder. Unfortunately, it doesn't contain the flag, but it does contain an interesting key file

<figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

This might be useful, so we'll keep in mind for later. Next, we can check the user's home directory, which contains a couple of interesting files

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Inspecting the files, we have a file containing the encrypted flag, as well as a script to get and encrypt the flag.&#x20;

{% code overflow="wrap" %}
```python
import os, socket
from urllib.request import urlopen

data = bytearray(urlopen("http://192.168.76.1:9000/flag.txt").read())
keys = [socket.gethostname().encode(), os.getcwd().encode(), open("/secret/key", "rb").read().strip()]
print(data, keys)
for k in keys:
    if k:
        for i in range(len(data)): data[i] ^= k[i % len(k)]

open("flag.enc", "w").write(data.hex())
```
{% endcode %}

To decrypt the file, we'll also need the hostname of the current system, and the the current working directory when running the command. To get the hostname, we can read the <mark style="color:red;">**/etc/hostname**</mark> file to get the hostname.

<figure><img src="../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

To get the current working directory, we can examine <mark style="color:red;">**.ash\_history**</mark>, which will contain the command history of the user.&#x20;

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

The user ran <mark style="color:red;">**cd \~**</mark>, which is used to change directory into the user's home directory (/home/jrjrjr). With all the necessary information gathered, we can write a Python script to decrypt the flag.

```python
import os
import socket

with open("flag.enc", "r") as f:
    data = bytearray.fromhex(f.read().strip())
keys = [
    b'revengeseeker',
    b'/home/jrjrjr',
    open("key", "rb").read().strip()
]
for k in keys:
    if k:
        for i in range(len(data)):
            data[i] ^= k[i % len(k)]
print("Flag:", data.decode(errors="replace"))
```

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>
