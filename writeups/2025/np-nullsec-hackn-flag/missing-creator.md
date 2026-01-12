---
description: 'Creator: Tofu.boi2'
---

# Missing Creator

## Description

{% code overflow="wrap" %}
```
RyanJohnJames was supposed to create this hard challenge, but he vanished before submitting it. The last thing he told me was that the flag was hidden at some paste site. Can you find the flag?
```
{% endcode %}

Category: OSINT (Hard)

## Walkthrough

We're not given with much, other than a person's (user)name. Putting this name into a search engine, we return with an interesting result

<figure><img src="../../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

The GitHub username matches the name provided in the description, so we can search around the account further. Looking through the account's past activity, I notice the following repository

<figure><img src="../../.gitbook/assets/image (219).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (220).png" alt=""><figcaption><p>Unpaid labour</p></figcaption></figure>

There is a README file, as well as a Python script. Unfortunately, there is only one line in the file

<figure><img src="../../.gitbook/assets/image (221).png" alt=""><figcaption><p>Could've at least left a cookie or something :(</p></figcaption></figure>

Since the file indicates that it was changed 4 days ago, we can look through the history of the file to see if the flag was stored here at some point in time

<figure><img src="../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

Looks like there is a password hash, as well as a partially corrupted URL, likely pointing to Pastebin. We can navigate to the URL, and insert in the provided endpoint to access it

<figure><img src="../../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>

The pastebin is encrypted, likely using the password that we saw earlier. We could try to crack the password ourselves using John the Ripper or Hashcat, but we have an easier method. There is an online resource called [Crackstation](https://crackstation.net) that allows us to match a specified password hash to massive pre-computed lookup tables to find a match

<figure><img src="../../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

In case you can't see because of the horrible colors, the password is "spencerfoleyroxmysox". Going back to the Pastebin, we can now unlock the note using our recovered password

<figure><img src="../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

## Conclusion

If you had done what I had did when solving the challenge for the first time, you may have uncovered the following earlier commit (commit 0a40b83)

<figure><img src="../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

Unfortunately, this password hash couldnt be cracked by Crackstation, and I wasnt interested in spending anywhere from 5 minutes to 23 hours trying to crack a possibly-uncrackable password or invalid hash.
