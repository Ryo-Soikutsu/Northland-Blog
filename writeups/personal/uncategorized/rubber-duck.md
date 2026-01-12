---
description: 'Creator: Ryo Soikutsu'
---

# Rubber Duck

## Description

{% code overflow="wrap" %}
```
One of the AI ship captains have detected the connection of an unknown device on the ship's network. We fear that it may be a spy device. Can you find out what it is?

Flag format: NYP{serial number of USB containing the wallpaper_time of wallpaper change (UTC time, 24h format)} E.g. NYP{AAA12341123_2023-10-30_20:00:00}

Google Drive link: https://drive.google.com/file/d/1XIgAsuj_tP8CsQwDRHExk3IAfPUMpjh7/view?usp=sharing
```
{% endcode %}

Category: Forensics

## Walkthrough

Downloading the provided file, we are met with an AD1 (AccessData format) file, which is commonly used for disk forensics. We can open the file in a tool called FTK Imager, which opens up a Windows-like filesystem

&#x20;

<figure><img src="../../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

The challenge wants us to find the following information

* The serial number of the USB containing the wallpaper
* The time of the wallpaper update

To find the first piece of information, we can look for the Partition Windows event logs, which will contain, among other things, connect and disconnect events for removable devices. These logs can be found in <mark style="color:red;">**`C:\Windows\System32\winevt\Logs`**</mark>, called <mark style="color:red;">**`Microsoft-Windows-Partition%4Diagnostic.evtx`**</mark>&#x20;

<figure><img src="../../.gitbook/assets/image (192).png" alt=""><figcaption></figcaption></figure>

We can export the file by right clicking it -> Export Files -> Browse For Folder (pick a folder to save the log file to), then open it using the built-in Windows Event Viewer utility

<figure><img src="../../.gitbook/assets/image (193).png" alt=""><figcaption></figcaption></figure>

To view additional information, we can click on the Details tab on the bottom center pane. This will contain more info about the removable device, including its serial number

<figure><img src="../../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>

Before continuing with the first step, we'll first have to find the next part of the flag (When the wallpaper was updated). To do that, we can navigate to the following directory (<mark style="color:red;">**`C:\Users\Solitude\AppData\Roaming\Microsoft\Windows\Themes`**</mark>) and look at the <mark style="color:red;">**`TranscodedWallpaper`**</mark> file

<figure><img src="../../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

The date modified indicates that the wallpaper was updated at 5:13:31pm. Now we can return to the event logs, and search for the removable device removed event with the closest timestamp

* Hint: To identify a removed event, you can look for events where the <mark style="color:red;">**`BytesPerSector`**</mark> and <mark style="color:red;">**`Capacity`**</mark> values are 0
* Also make sure to compare the <mark style="color:red;">**`TimeCreated`**</mark>\\<mark style="color:red;">**`SystemTime`**</mark>  attribute, which will show the timestamp that was recorded in the event log, accounting for the timezone data that was set on the machine being investigated

<figure><img src="../../.gitbook/assets/image (196).png" alt=""><figcaption><p>The closest timestamp to the update timestamp</p></figcaption></figure>

Now that we have found the removable device, we can scroll down to the <mark style="color:red;">**`SerialNumber`**</mark> attribute, copy it and insert it along with the timestamp that we found earlier for part 2 into the final flag

<figure><img src="../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

This was a bit of a scuffed challenge, as I did not account for the fact that participants may use other tools to access the AD1 image. There was feedback of some participants who used [AD1-tools](https://github.com/al3ks1s/AD1-tools), and were unable to access the image

There were also a few assumptions that were made for the challenge

* The wallpaper update timestamp is the same, or very close to the disconnect time of the removable device
* The timezone given was in the correct format

This challenge was also inspired by a challenge from Brainhack 2025 CDDC qualifier round, called Assignment Done. Participants who have attempted or solved the mentioned challenge would have a significantly easier time with this challenge

For now, enjoy a picture of my birb

<figure><img src="../../.gitbook/assets/image (198).png" alt=""><figcaption><p>Comically large birb</p></figcaption></figure>
