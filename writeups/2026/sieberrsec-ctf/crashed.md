---
description: 'Category: Forensics (Easy)'
---

# Crashed

## Description

{% code overflow="wrap" %}
```
My friends keep pressing my power button, this time my whole pc shut down while I was doing my practice paper :( Help me recover it please

Get dist crashed.E01 from https://mega.nz/file/7uJUCYKA#bM_diIYfKh7qF5yGNEkHl2DtEnSiVg5bvNMn4EZEYsc
```
{% endcode %}

## Walkthrough

We're provided with an E01 disk image file, which we can open in FTK Imager for analysis.

<figure><img src="../../.gitbook/assets/image (427).png" alt=""><figcaption></figcaption></figure>

Based on the description provided, the text editor/application likely entered a crash state, and cached the unsaved file somewhere on disk. Initially, I tried to search in the user's home directory, since Windows apps typically store appdata in the user directory's AppData folder.

<figure><img src="../../.gitbook/assets/image (428).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (429).png" alt=""><figcaption></figcaption></figure>

Unfortunately they're all empty, so I started to search elsewhere. Asking an LLM for assistance doesnt yield any useful information either

<figure><img src="../../.gitbook/assets/image (430).png" alt=""><figcaption></figcaption></figure>

I started looking through the usual directories where applications store data/configs (/etc, /var, /tmp), but couldnt find anything either. After doing a bit more Googling, I discovered that Linux systems have a special directory called Lost+Found, which can be seen here

<figure><img src="../../.gitbook/assets/image (431).png" alt=""><figcaption></figcaption></figure>

I stumbled on an old forums post asking about this directory, and this was the explanation provided for the directory

<figure><img src="../../.gitbook/assets/image (432).png" alt=""><figcaption></figcaption></figure>

This folder basically contains files that the operating system deems are actually files, but were never saved properly to the filesystem, either due to hardware or software faults (like a sudden power cut). If we look inside this folder, we find a single entry containing a script

<figure><img src="../../.gitbook/assets/image (433).png" alt=""><figcaption></figcaption></figure>

Sure enough, the orphaned text file contains our flag, as well as an answer that confirms the forums answer earlier.

<figure><img src="../../.gitbook/assets/image (434).png" alt=""><figcaption></figcaption></figure>
