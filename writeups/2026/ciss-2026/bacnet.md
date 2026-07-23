---
description: 'Category: BACnet'
---

# BACnet

## Description

{% code overflow="wrap" %}
```
Please refer to BACnet_dataset.pcap for the rest of the seven (7) BACnet category questions

MD5 Hash of PCAP: 84EFBB4ADDEE2E1C554F4537E2E49E84

Scenario: A dodgy pirate tried to pay off his debt to you with a potential "treasure map". He claims that this "treasure map" would have cost an arm or a leg, if you tried to buy it from an initial access broker! After you squeeze him for a better deal, he reveals that the " treasure map" is really a .pcap file...

Specifically, it is a record of the traffic within a network, that someone managed to sniff and capture, over some period of time. Before you decide whether the network is worth the effort to attack, you decide to analyze the pcap file, to determine the "lay of the land"...

The following questions in the BACnet category just might be some of the questions worth asking?
```
{% endcode %}

(Note that this page is the combined writeup for all the challenges in the BACnet category)

## Walkthrough

We're provided with a simple network capture file containing a large volume of BACnet packets. Due to the unique challenge structure, we will be covering each question one by one

<figure><img src="../../.gitbook/assets/image (471).png" alt=""><figcaption></figcaption></figure>

### Question 1 - What type of OT/ICS system most likely generated the .pcap file?

We can search for I-Am Responses, which contains a large amount of valuable information. Notably, they contain the device name (also called the object identifier) and the vendor ID, which can be used to identify the type of OT system running. Clicking on the first I-Am packet, we see the following data

<figure><img src="../../.gitbook/assets/image (472).png" alt=""><figcaption></figcaption></figure>

The OT system running is labelled as Servisys, Inc. A quick Google search tells us that this company "specializes in automatic control solutions and HVAC equipment maintenance"

<figure><img src="../../.gitbook/assets/image (473).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (474).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (475).png" alt=""><figcaption></figcaption></figure>

### Question 2 - Which IP has a CO2 Sensor?

(Note: This challenge wasnt solved during the course of the CTF)

We can just search for "CO2" in Wireshark

<figure><img src="../../.gitbook/assets/image (494).png" alt=""><figcaption></figcaption></figure>

We only get one address, 172.120.236.61.&#x20;

### Question 3 - Which IP is most likely the BACNet client/browser?

To look for the client, we can simply look for a common IP address that appears a disproportionate amount of times in comparison to the other IP addresses. Looking through the logs, we notice that 172.120.236.70 seems to keep appearing over and over again. In addition, its the only address that consistently sends request packets to the other addresses, which can indicate automated enumeration of devices by a BACNet browser.

<figure><img src="../../.gitbook/assets/image (476).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (477).png" alt=""><figcaption></figcaption></figure>

### Question 4 - Which IP is most likely the weather station?

We can search for the word "Weather" in Wireshark

<figure><img src="../../.gitbook/assets/image (478).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (479).png" alt=""><figcaption></figcaption></figure>

This acknowledgement packet contains the string "WeatherStation", which strongly suggests that the IP address 172.120.236.63 is the weather station we're looking for

<figure><img src="../../.gitbook/assets/image (480).png" alt=""><figcaption></figcaption></figure>

### Question 5 - What are the overall trends in the change in temperature for the two devices

This question asks us to read the changes in a common variable between two devices, at 172.120.236.62 and 172.120.236.63.

<figure><img src="../../.gitbook/assets/image (481).png" alt=""><figcaption></figcaption></figure>

We can use the following query to return only packets containing the specified object identifier

<figure><img src="../../.gitbook/assets/image (482).png" alt=""><figcaption></figcaption></figure>

Start at the first few packets, take note of the current value of the object identifier, and then scroll to the end and check again

<figure><img src="../../.gitbook/assets/image (483).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (484).png" alt=""><figcaption></figcaption></figure>

Looking at the values for 172.120.236.62, we can see that the temperature decreases over time. We can repeat this for .63, which should increase over time based on our observations

<figure><img src="../../.gitbook/assets/image (485).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (486).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (487).png" alt=""><figcaption></figcaption></figure>

### Question 6 - For the devices with the most object identifiers, which device has the highest amount of analog inputs?

We can start by searching for acknowledgement packets (Type 3) with a property identifier of 76 (representing the string "object-list"). We can then go through the returned list of objects, and count the number of identifiers

<figure><img src="../../.gitbook/assets/image (489).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (488).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (490).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (492).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (495).png" alt=""><figcaption></figcaption></figure>

### Question 7 - In the whole dataset, how many BACnet binary outputs are there in total, and how many unique IPs have at least one BACnet binary output?

Refer to question 6

<figure><img src="../../.gitbook/assets/image (493).png" alt=""><figcaption></figcaption></figure>
