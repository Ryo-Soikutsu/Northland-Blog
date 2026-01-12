---
description: 'Solved by: Ryo Soikutsu'
---

# Heartbleed

## Challenge Description

```
While working on a server, the file flag.html was accidentally deleted.
Luckily, a previously saved network capture contains the file.
Your task is to extract flag.html from the provided .pcap file.
```

Category: Crypto (Unrated)

## Walkthrough

We are provided with a webpage and a network capture file. When accessing the website, we see the following

<figure><img src="../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

As hinted by the challenge name, this website is vulnerable to the Heartbleed vulnerability (CVE-2014-0160), a critical flaw in the OpenSSL cryptographic library used to implement TLS/SSL encryption on HTTPS websites. Heartbleed allows attackers to exploit a flaw in the heartbeat extension of OpenSSL, enabling them to read sensitive memory contents from the server, including private keys, usernames, passwords, and other confidential data.

Switching focus to the network capture, we are greeted with encrypted traffic

<figure><img src="../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

The web traffic has been encrypted, and we cannot view the flag.html page specified in the challenge directly. In order to do so, we have to first extract the private key used to encrypt the traffic. We can do that by using a publicly-available POC script (link [here](https://github.com/sensepost/heartbleed-poc)) and running it against the target website

<figure><img src="../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

As shown, the exploit was successful, and we managed to dump out the private key on the first attempt. The script saves any memory recovered from the exploit in a dump.bin file, which we can output in a terminal to view the full key

* Note that you may sometimes need to run the exploit with multiple requests (using the --num flag) to get the private key

<figure><img src="../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

We can save this private key and pass it to Wireshark, which will help us decrypt the TLS traffic

<figure><img src="../../.gitbook/assets/image (99).png" alt=""><figcaption><p>Excuse the bad censorship :P</p></figcaption></figure>

Like that, the once-encrypted traffic is now decrypted, and its contents in plain view for us to see

<figure><img src="../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (102).png" alt=""><figcaption><p>Final flag</p></figcaption></figure>

## Conclusion

I had originally been stumped on this challenge, as I thought we had to directly extract the private key from the provided traffic somehow. I only decided to try out running the POC after some further Googling, and finding out about the exploit script

(Could've solved it on the 1st day, but then again that meant that I would have 0 solves on day 2 :P)
