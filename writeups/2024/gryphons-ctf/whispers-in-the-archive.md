---
description: 'Solved by: Ryo Soikutsu'
---

# Whispers in the Archive

## Challenge Description

{% code overflow="wrap" %}
```
A mysterious archive has arrived. Can you uncover all the secrets it holds, including the well-hidden 
flags scattered throughout its concealed depths?
```
{% endcode %}

Category: Forensics (Medium)

## Walkthrough

Downloading the file, we see that we get a dist.tar file. Naturally, I tried to open the tar file using an unzipper, but it couldn't be uncompressed. Then, I tried to open it in FTK Imager, thinking it was a disk image, which succeeded.

<figure><img src="../../.gitbook/assets/image (166).png" alt=""><figcaption><p>Add Evidence Item > Image File</p></figcaption></figure>

The disk image was of a Linux server, so I automatically started searching in the /var, /home, /root and /tmp directories, since they tend to have something interesting in them.

In /root/.bashrc, we find the first flag fragment

<figure><img src="../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

The second flag fragment was found in /root/.viminfo, which indicated that there was a file named /bin/hello\_world.py

<figure><img src="../../.gitbook/assets/image (168).png" alt=""><figcaption></figcaption></figure>

Navigating to the file, we get the following code

```python
import codecs
import base64

encoded_flag = "XzFhX1N2eTMzbGZHM3pfZTF0dTF9" # Base64 encoded and ROT13'd

def decode_flag(encoded_str):
    try:
        base64_decoded = base64.b64decode(encoded_str).decode('utf-8')
        rot13_decoded = codecs.decode(base64_decoded, 'rot_13')
        return rot13_decoded
    except (base64.binascii.Error, Exception) as e:
        print("Error decoding: {}".format(e))  # Use .format() for older Python
        return None

decoded_part = decode_flag(encoded_flag)

if decoded_part:
   print(decoded_part)
```

Running this in WSL we get the next fragment

<figure><img src="../../.gitbook/assets/image (169).png" alt=""><figcaption></figcaption></figure>

The final flag fragment was located in /tmp/.secret, and could actually be found by using the strings and grep commands on the image

<figure><img src="../../.gitbook/assets/image (170).png" alt=""><figcaption><p>Recovering using FTK Imager</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (171).png" alt=""><figcaption><p>"Cheesing" using strings and grep</p></figcaption></figure>

With all 3 fragments, we can reconstruct the flag and submit it as the answer
