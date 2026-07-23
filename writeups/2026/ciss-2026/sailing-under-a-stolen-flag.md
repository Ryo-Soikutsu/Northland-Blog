---
description: 'Category: Maritime OT'
---

# Sailing Under a Stolen Flag

## Description

{% code overflow="wrap" %}
```
Port security has raised an alert. During a routine sweep of AIS traffic, harbour records show that one vessel's electronic signature appears under two entirely different ship names. Someone is broadcasting a stolen identity. Identify the vessel that has been cloned.

What is the shared registration number broadcast by both vessels?

Submit the flag as CISS26{number}.
```
{% endcode %}

## Walkthrough

We're provided with a large network capture file, with a huge amount of UDP packets.

<figure><img src="../../.gitbook/assets/image (466).png" alt=""><figcaption></figcaption></figure>

In the packet preview window, we notice a suspicious string of text inside. If we follow the UDP stream, we can see the full conversation going on.

<figure><img src="../../.gitbook/assets/image (467).png" alt=""><figcaption></figcaption></figure>

Some Googling (or asking an LLM) will identify this data format as NMEA 0183, which is commonly used by marine electronics to share navigation data, such as GPS coordinates and ship identification information. After a little more searching around, we can find a webpage detailing the specifications of the AIVDM/AIVDO message formats, which contain the identification information that we're interested in (link to webpage [here](https://gpsd.gitlab.io/gpsd/AIVDM.html)).

<details>

<summary>AIVDM/AIVDO Message Structure</summary>

Example message (each field is comma delimited, with the exception of the asterisk suffix)

```
!AIVDM,1,1,,B,177KQJ5000G?tO`K>RA1wUbN0TKH,0*5C
```

* Field 1 (AIVDM) - Identifies the message as AIVDM format
* Field 2 (1) - Identifies the number of fragments in one message (some message types, such as Type 5, are too large to fit in one message, so it is fragmented into separate messages instead)
* Field 3 (1) - Identifies the current fragment number of this message
* Field 4 (<mark style="color:$danger;">**NULL**</mark>) - Identifies the sequential message ID&#x20;
* Field 5 (B) - Identifies the radio channel frequency/code in use
* Field 6 (177...TKH) - Encoded data payload
* Field 7 (0) - Number of padding bytes in the data payload to reach the 6-bit boundary
* Checksum (5C) - The checksum of the message, calculated by XOR'ing all the fields together

</details>

To solve this challenge, we will have to take advantage of Type 5 messages, which contain a wealth of identification information. Notably, it provides us with the following values

* MMSI - Maritime Mobile Ship Identifier, functions like an international phone number
* IMO ID - Permanent identifier assigned to all ships, used to track ships
* Callsign
* Ship name

We can refer to the documentation to figure out the format of these types of messages

<figure><img src="../../.gitbook/assets/image (468).png" alt=""><figcaption></figcaption></figure>

### Writing the Script

We can start by writing a function to load the capture, and extract all packets containing the AIVDM or AIVDO header

{% code overflow="wrap" %}
```python
def dump_pcap(pcap_path): 
	payloads = []
	capture = rdpcap(pcap_path)
	print("Pcap file loaded")
	for pkt in tqdm(capture):
		if pkt.haslayer(Raw):
			data = bytes(pkt[Raw])
			text = data.decode('ascii', errors='ignore')
			for marker in ("!AIVDM","!AIVDO"):
				idx = text.find(marker)
				if idx != -1:
					payloads.append(text[idx:].split('\r')[0].split('\n')[0])

	return payloads
```
{% endcode %}

AIVDM/O data payloads are encoded, so we will have to unarmor and decode them to their binary format. To recover the original binary data, we need to subtract 48 from the ASCII character value. If the value is higher than 40, we subtract an additional 8. This will give us the unarmored decimal value, which we can then convert to bits directly. It is important to ensure that each binary string is padded to the 6-bit boundary, otherwise it will cause misclassifications and other parsing issues later on.

{% code overflow="wrap" %}
```python
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
```
{% endcode %}

There may be other Types in the network capture other than Type 5, so we should filter them out. While we're at it, we might as well parse the Type 5 messages that we find, and extract the values that we're interested in

{% code overflow="wrap" %}
```python
def parse_type5(bits):
	msg_type = bits[0:6]
	if msg_type != '000101':
		return None

	mmsi = int(bits[8:38],2)
	imo = int(bits[40:70],2)
	callsign = decode_string(bits[70:112])
	shipname = decode_string(bits[112:232])
	return {"mmsi":mmsi,"imo":imo,"callsign":callsign,"shipname":shipname}
```
{% endcode %}

We almost have everything we need to get our flag. Now, we need to create dictionaries that will allow us to find MMSI or IMO IDs with multiple callsigns.

{% code overflow="wrap" %}
```python
by_mmsi = defaultdict(set)
by_imo = defaultdict(set)

for sentence in valid:
	bits = convert_payload(sentence.split(',')[5])
	rec = parse_type5(bits)
	if rec and rec["shipname"]:
		by_mmsi[rec["mmsi"]].add(rec["shipname"])
		if rec["imo"] != 0:
			by_imo[rec["imo"]].add(rec["shipname"])

print("MMSIs broadcasting more than one name:")
for mmsi, names in by_mmsi.items():
	if len(names) > 1:
		print(f"MMSI {mmsi}: {names}")

print("IMO numbers broadcasting more than one name:")
for imo, names in by_imo.items():
	if len(names) > 1:
		print(f"IMO {imo}: {names}")
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

def parse_type5(bits):
	msg_type = bits[0:6]
	if msg_type != '000101':
		return None

	mmsi = int(bits[8:38],2)
	imo = int(bits[40:70],2)
	callsign = decode_string(bits[70:112])
	shipname = decode_string(bits[112:232])
	return {"mmsi":mmsi,"imo":imo,"callsign":callsign,"shipname":shipname}

pcap_path = "c1_mmsi_collision.pcap"
raw = dump_pcap(pcap_path)
print(f"Found {len(raw)} potential AIVDM messages")

print("Validating potential AIVDM messages...")
valid = validate_vdm_msg(raw)
print(f"Validated {len(valid)} messages")
print("Displaying first and last 5 valid messages")
print("\n".join(valid[:5]))
print("="*25)
print("\n".join(valid[-5:-1]))

by_mmsi = defaultdict(set)
by_imo = defaultdict(set)

for sentence in valid:
	bits = convert_payload(sentence.split(',')[5])
	rec = parse_type5(bits)
	if rec and rec["shipname"]:
		by_mmsi[rec["mmsi"]].add(rec["shipname"])
		if rec["imo"] != 0:
			by_imo[rec["imo"]].add(rec["shipname"])

print("MMSIs broadcasting more than one name:")
for mmsi, names in by_mmsi.items():
	if len(names) > 1:
		print(f"MMSI {mmsi}: {names}")

print("IMO numbers broadcasting more than one name:")
for imo, names in by_imo.items():
	if len(names) > 1:
		print(f"IMO {imo}: {names}")
```
{% endcode %}

</details>

We can run this script, which will give us a duplicate MMSI with multiple unique names

<figure><img src="../../.gitbook/assets/image (469).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (470).png" alt=""><figcaption></figcaption></figure>

