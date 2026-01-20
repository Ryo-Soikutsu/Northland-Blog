---
description: 'Creator: Jun Wei'
---

# Beaconing

## Description

{% code overflow="wrap" %}
```
An IT staff on the internal network (10.0.0.0/24) reported unauthorised data transfers! We managed to capture some network traffic from the affected segment, and initial analysis suggests that a host was compromised via a payload download and is now maintaining a persistent Command-and-Control (C2) connection...

Your task is to perform further investigation. You should identify 2 crucial indicators, which are the C2 IP address and the beacon interval. Do note that the beacon interval should be a whole number i.e. no decimal places.

The flag should be submitted in the following format: HNF25{IP_Interval} e.g. HNF25{1.2.3.4_20}
```
{% endcode %}

Category: Forensics (Medium)

## Walkthrough

We are provided with a network capture file containing some HTTP, ICMP and TCP traffic

<figure><img src="../../.gitbook/assets/image (254).png" alt=""><figcaption></figcaption></figure>

Right away, we see a suspicious endpoint (/payload.exe) being queried over HTTP. Inspecting the HTTP stream, we can see that this is a large stream of binary data (likely to simulate a C2 beacon or shell being transferred)

<figure><img src="../../.gitbook/assets/image (255).png" alt=""><figcaption></figcaption></figure>

Scrolling to the third data stream, we notice suspicious check-in requests to a remote server

<figure><img src="../../.gitbook/assets/image (256).png" alt=""><figcaption></figcaption></figure>

This traffic is done by a beacon program on the victim server. We can switch over to the main Wireshark window, and look at the IP addresses participating in this data stream

<figure><img src="../../.gitbook/assets/image (257).png" alt=""><figcaption></figcaption></figure>

Since we've already been told that the internal network is on the 10.0.0.0/24 subnet, it means that the C2 server has to be at 93.184.216.34. To get the beacon interval, we can take any two timestamps (preferably in "Seconds since first captured packet") and subtract them to get the interval period
