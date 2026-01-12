---
description: 'Creator: scuffed'
---

# ping ping ping

## Description

{% code overflow="wrap" %}
```
We managed to get a forensics dump of a cybercriminal's laptop. We know they used Discord for communications, but we can't find a trace of Discord on their laptop! What we know is they definitely had notifications enabled...
```
{% endcode %}

Category: Forensics (Hard)

## Walkthrough

We are provided with a disk image, which we can open in FTK Imager to view it in a more human-friendly format

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Based on the description, we are supposed to be looking for Discord artifacts. However, searching through the usual directories associated with the application, we don't find anything.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Likely, the author used the web version of Discord, or one of the portable versions of Discord available online. The description also hints to look at notifications, so we can try to search for artifacts related to Windows notifications. These notifications are persistent, not being removed after deletion of the original application. Windows will store these notifications in SQLite databases under each individual user profile, which is located at the following path

* <mark style="color:red;">%LOCALAPPDATA%\Microsoft\Windows\Notifications\wpndatabase.db</mark>

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

We can export the database file to our host machine, and open it up in SQLitebrowser.&#x20;

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

The 5th notification contains a Base64-encoded string, which we can copy out and decode

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## Conclusion

This was a pretty interesting disk forensics challenge overall, excusing the slight mistake made by the author at the start (the wpndatabase file did not correctly have the Discord notifications contained inside).&#x20;

Also Im very proud of my first blood :fire:

<figure><img src="../../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>
