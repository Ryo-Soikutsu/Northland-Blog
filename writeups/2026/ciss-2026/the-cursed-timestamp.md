---
description: 'Category: Maritime OT'
---

# The Cursed Timestamp

## Description

{% code overflow="wrap" %}
```
The Black Pearl's navigation logs appear to be cursed. Most GPS date/time records follow the expected timeline — but one is from another time entirely. One GPZDA timestamp is anomalous. What date does it show?

Submit the flag as CISS26{DDMMYYYY}.

MD5 Checksum of PCAP file: b325c65da204a8e9353860d6f0a0505e
```
{% endcode %}

## Walkthrough

## Walkthrough

We're provided with a large network capture file, with a huge amount of UDP packets.

<figure><img src="../../.gitbook/assets/image (466).png" alt=""><figcaption></figcaption></figure>

In the packet preview window, we notice a suspicious string of text inside. If we follow the UDP stream, we can see the full conversation going on.

<figure><img src="../../.gitbook/assets/image (467).png" alt=""><figcaption></figcaption></figure>

Some Googling (or asking an LLM) will identify this data format as NMEA 0183, which is commonly used by marine electronics to share navigation data, such as GPS coordinates and ship identification information. After a little more searching around, we can find a webpage detailing the specifications of the GPZDA message format, which contain the identification information that we're interested in (link to webpage [here](https://gpsd.gitlab.io/gpsd/AIVDM.html)).

The message structure of GPZDA messages is very simple, and doesn't require much advanced parsing like the previous NMEA challenges. We can copy the script used in a [previous challenge](https://blog.northland.dev/writeups/2026/ciss-2026/sailing-under-a-stolen-flag), and tweak it a little to parse this new message.

{% code overflow="wrap" %}
```python
def parse_gpzda(message):
	body = message.strip().lstrip('$').split('*')[0]
	tokens = body.split(",")
	time_str,day,month,year,zone_hr,zone_min = tokens[1:7]
	hh = int(time_str[0:2])
	mm = int(time_str[2:4])
	ss = float(time_str[4:])
	microsec = int(round((ss - int(ss) * 1000000)))


	dt_utc = datetime(int(year),int(month),int(day),hh,mm,int(ss),microsec,tzinfo=timezone.utc)
	return [dt_utc,zone_hr,zone_min]
	
...
print("Displaying GPZDA datetime data")
print("="*25)
for msg in raw:
	dt_utc,zone_hr,zone_min = parse_gpzda(msg)
	print(f"Datetime: {dt_utc} - Zone delta: {zone_hr}/{zone_min}")
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
from datetime import *

def validate_vdm_msg(payloads):
	valid = []
	for payload in payloads:
		body, _, chk = payload.lstrip('$').partition('*')
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
			idx = text.find("$GPZDA")
			if idx != -1:
				payloads.append(text[idx:].split('\r')[0].split('\n')[0])

			"""
			for marker in ("$GPZDA"):
				idx = text.find(marker)
				if idx != -1:
					payloads.append(text[idx:].split('\r')[0].split('\n')[0])
			"""

	return payloads

def parse_gpzda(message):
	body = message.strip().lstrip('$').split('*')[0]
	tokens = body.split(",")
	time_str,day,month,year,zone_hr,zone_min = tokens[1:7]
	hh = int(time_str[0:2])
	mm = int(time_str[2:4])
	ss = float(time_str[4:])
	microsec = int(round((ss - int(ss) * 1000000)))


	dt_utc = datetime(int(year),int(month),int(day),hh,mm,int(ss),microsec,tzinfo=timezone.utc)
	return [dt_utc,zone_hr,zone_min]

pcap_path = "capture.pcap"
raw = dump_pcap(pcap_path)
print(f"Found {len(raw)} potential GPZDA messages")

print("Validating potential GPZDA messages...")
valid = validate_vdm_msg(raw)
print(f"Validated {len(valid)} messages")
print("Displaying first and last 5 valid messages")
print("\n".join(valid[:5]))
print("="*25)
print("\n".join(valid[-5:-1]))

print("Displaying GPZDA datetime data")
print("="*25)
for msg in raw:
	dt_utc,zone_hr,zone_min = parse_gpzda(msg)
	print(f"Datetime: {dt_utc} - Zone delta: {zone_hr}/{zone_min}")
```
{% endcode %}

</details>

<figure><img src="../../.gitbook/assets/image (496).png" alt=""><figcaption></figcaption></figure>

Our answer is the timestamp that looks almost similar to the rest of the outputs

<figure><img src="../../.gitbook/assets/image (497).png" alt=""><figcaption><p>Can you spot the imposter?</p></figcaption></figure>
