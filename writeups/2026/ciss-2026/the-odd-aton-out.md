---
description: 'Category: Maritime OT'
---

# The Odd AtoN Out

## Description

{% code overflow="wrap" %}
```
PCAP Used: chall2.pcap

MD5 Hash of PCAP: E6A6D9163A4FBDA92D2BBB3249A182AA

This area has AtoNs to help us navigate the busy waters, but something seems off.

Submit the coordinates of the AtoN that looks off.

Provide the flag in this format CISS26{latitude,longitude}
```
{% endcode %}

## Walkthrough

We're provided with a large network capture file, with a huge amount of UDP packets.

<figure><img src="../../.gitbook/assets/image (466).png" alt=""><figcaption></figcaption></figure>

In the packet preview window, we notice a suspicious string of text inside. If we follow the UDP stream, we can see the full conversation going on.

<figure><img src="../../.gitbook/assets/image (467).png" alt=""><figcaption></figcaption></figure>

Some Googling (or asking an LLM) will identify this data format as NMEA 0183, which is commonly used by marine electronics to share navigation data, such as GPS coordinates and ship identification information. After a little more searching around, we can find a webpage detailing the specifications of the AIVDM/AIVDO message formats, which contain the identification information that we're interested in (link to webpage [here](https://gpsd.gitlab.io/gpsd/AIVDM.html)).

Specifically, we're interested in Type 21 messages, which provide information about Aid-to-Navigation (AtoN) objects out at sea. We're interested in the following values

* MMSI
* Name (of AtoN)
* Longitude
* Latitude

To save us some time, we can repurpose the script from a [previous challenge](https://blog.northland.dev/writeups/2026/ciss-2026/sailing-under-a-stolen-flag), and make appropriate modifications to parse Type 21 messages instead

{% code overflow="wrap" %}
```python
def decode_signed(bits):
	val = int(bits,2)
	if bits[0] == '1':
		val -= (1 << len(bits))
	return val

def parse_type21(bits):
	msg_type = bits[0:6]
	if msg_type != '010101':
		return None

	if len(bits) < 271:
		return None

	mmsi = int(bits[8:38],2)
	aid_type = int(bits[38:43],2)
	shipname = decode_string(bits[43:163])
	longitude = decode_signed(bits[164:192]) / 600000.0
	latitude = decode_signed(bits[192:219]) / 600000.0
	virtual = int(bits[269:270],2)
	return {
		"mmsi":mmsi,
		"aid_type":aid_type,
		"shipname":shipname,
		"longitude":longitude,
		"latitude":latitude,
		"virtual":virtual
	}
	
...
virtual_aton = defaultdict(list)
real_aton = defaultdict(list)
records = []

for sentence in valid:
	bits = convert_payload(sentence.split(',')[5])
	fill_bits = int(sentence.lstrip('!').partition('*')[0].split(',')[6])
	if fill_bits:
		bits = bits[:-fill_bits]
	rec = parse_type21(bits)
	if rec is None:
		continue
	if rec["virtual"] == 0:
		if rec["mmsi"] in real_aton:
			continue
		real_aton[rec["mmsi"]].append(rec)
	else:		
		if rec["mmsi"] in virtual_aton:
			continue
		virtual_aton[rec["mmsi"]].append(rec)

print("Displaying all real AToNs:")
print("="*25)
for mmsi,rec_list in real_aton.items():
	for rec in rec_list:
		print(f"MMSI {mmsi} - Ship name: {rec["shipname"]} / Longitude: {rec["longitude"]} / Latitude: {rec["latitude"]}")

print("Displaying all virtual AToNs:")
print("="*25)
for mmsi,rec_list in virtual_aton.items():
	for rec in rec_list:
		print(f"MMSI {mmsi} - Ship name: {rec["shipname"]} / Longitude: {rec["longitude"]} / Latitude: {rec["latitude"]}")
```
{% endcode %}

<details>

<summary>Final script</summary>

{% code overflow="wrap" %}
```python
"""
Reference: https://gpsd.gitlab.io/gpsd/AIVDM.html
"""

from scapy.all import rdpcap, Raw
from tqdm import *
from collections import defaultdict

def validate_vdm_msg(payloads):
	valid = []
	for payload in payloads:
		body, _, chk = payload.lstrip('!').partition('*')
		calc = 0
		for ch in body:
			calc ^= ord(ch)
		if f"{calc:02X}" == chk.strip().upper():
			valid.append(payload)
	return valid

def dump_pcap(pcap_path): 
	payloads = []
	print("Parsing pcap file...")
	capture = rdpcap(pcap_path)
	print("Finished parsing pcap file!")
	for pkt in tqdm(capture):
		if pkt.haslayer(Raw):
			data = bytes(pkt[Raw])
			text = data.decode('ascii', errors='ignore')
			for marker in ("!AIVDM","!AIVDO"):
				idx = text.find(marker)
				if idx != -1:
					payloads.append(text[idx:].split('\r')[0].split('\n')[0])

	return payloads

def convert_payload(payload):
	s1_decoded = []
	output = ""
	# Unarmor payload first
	for char in payload:
		decoded = ord(char) - 48
		if decoded > 40:
			decoded -= 8
		s1_decoded.append(decoded)

	# Convert to binary
	for char in s1_decoded:
		output += format(char,'06b')
	return output

def group_fragments(valid):
	groups = defaultdict(dict)
	singles = []
	for msg in valid:
		fields = msg.split(",")
		frag_count = int(fields[1])
		frag_num = int(fields[2])
		seq_id = fields[3]
		payload = fields[5]
		if frag_count == 1:
			singles.append(payload)
		else:
			groups[seq_id][frag_num] = payload

	reassembled = list(singles)
	for seq_id,frags in groups.items():
		if len(frags) == max(frags.keys()):
			ordered = "".join(frags[i] for i in sorted(frags.keys()))
			reassembled.append(ordered)
	return reassembled

def decode_string(bits):
	SIXBIT_CHARS = "@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\\]^_ !\"#$%&'()*+,-./0123456789:;<=>?"
	chars = []
	for i in range(0, len(bits) - 5, 6):
		chunk = bits[i:i+6]
		if len(chunk) < 6:
			break
		val = int(chunk, 2)
		chars.append(SIXBIT_CHARS[val])
	s = "".join(chars)
	return s.split("@")[0].rstrip()

def decode_signed(bits):
	val = int(bits,2)
	if bits[0] == '1':
		val -= (1 << len(bits))
	return val

def parse_type21(bits):
	msg_type = bits[0:6]
	if msg_type != '010101':
		return None

	if len(bits) < 271:
		return None

	mmsi = int(bits[8:38],2)
	aid_type = int(bits[38:43],2)
	shipname = decode_string(bits[43:163])
	longitude = decode_signed(bits[164:192]) / 600000.0
	latitude = decode_signed(bits[192:219]) / 600000.0
	virtual = int(bits[269:270],2)
	return {
		"mmsi":mmsi,
		"aid_type":aid_type,
		"shipname":shipname,
		"longitude":longitude,
		"latitude":latitude,
		"virtual":virtual
	}

pcap_path = "capture.pcap"
raw = dump_pcap(pcap_path)
print(f"Found {len(raw)} potential AIVDM messages")

print("Validating potential AIVDM messages...")
valid = validate_vdm_msg(raw)
print(f"Validated {len(valid)} messages")
print("Displaying first and last 5 valid messages")
print("\n".join(valid[:5]))
print("="*25)
print("\n".join(valid[-5:-1]))

virtual_aton = defaultdict(list)
real_aton = defaultdict(list)
records = []

for sentence in valid:
	bits = convert_payload(sentence.split(',')[5])
	fill_bits = int(sentence.lstrip('!').partition('*')[0].split(',')[6])
	if fill_bits:
		bits = bits[:-fill_bits]
	rec = parse_type21(bits)
	if rec is None:
		continue
	if rec["virtual"] == 0:
		if rec["mmsi"] in real_aton:
			continue
		real_aton[rec["mmsi"]].append(rec)
	else:		
		if rec["mmsi"] in virtual_aton:
			continue
		virtual_aton[rec["mmsi"]].append(rec)

print("Displaying all real AToNs:")
print("="*25)
for mmsi,rec_list in real_aton.items():
	for rec in rec_list:
		print(f"MMSI {mmsi} - Ship name: {rec["shipname"]} / Longitude: {rec["longitude"]} / Latitude: {rec["latitude"]}")

print("Displaying all virtual AToNs:")
print("="*25)
for mmsi,rec_list in virtual_aton.items():
	for rec in rec_list:
		print(f"MMSI {mmsi} - Ship name: {rec["shipname"]} / Longitude: {rec["longitude"]} / Latitude: {rec["latitude"]}")
```
{% endcode %}

</details>

<figure><img src="../../.gitbook/assets/image (498).png" alt=""><figcaption></figcaption></figure>

Most of the returned AtoN coordinates are rounded to 5 decimal places, except for Fort Canning Lightho(use). In addition, we can perform a Google search, which will tell us that Fort Canning Lighthouse was decommissioned a while ago, and isn't in use today.

> #### The Lighthouse today <a href="#the_lighthouse_today" id="the_lighthouse_today"></a>
>
> Due to its historic significance, a replica of the Lighthouse was later erected near its original site by the [government of Singapore](https://en.wikipedia.org/wiki/Government_of_Singapore).[<sup>\[6\]</sup>](https://en.wikipedia.org/wiki/Fort_Canning_Lighthouse#cite_note-6) Though its lights are fully functional, it no longer serves its original purpose but stands today as a legacy of its contribution to Singapore maritime history.
>
> * [Wikipedia](https://en.wikipedia.org/wiki/Fort_Canning_Lighthouse)

For longitude and latitude coordinates, they are typically rounded to 5 or 6 decimal places.
