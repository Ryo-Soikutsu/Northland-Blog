---
description: 'Solved by: Ryo Soikutsu'
---

# EZ

## Challenge Description

{% code overflow="wrap" %}
```
Sometimes, the simplest puzzles hold the biggest secrets so don’t overthink it. Somewhere in the given data, a hidden message is waiting to be uncovered. Whether it’s a classic cipher, a simple encoding, or just a clever twist, your task is to decrypt the message and reveal its true meaning.

Think fast, stay sharp, and remember: the answer is often hiding in plain sight.
```
{% endcode %}

Category: Cryptography (Easy)&#x20;

## Walkthrough

Unzipping the 7zip file given, we discover a flag.enc file, supposedly containing an encrypted form of the flag. Running the file command on the encrypted file indicates that it is a Gzip-compressed file.

<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

Using CyberChef, we can unzip the file and decode it to get the flag

<figure><img src="../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

## Conclusion

Not much to say here, quick and easy challenge
