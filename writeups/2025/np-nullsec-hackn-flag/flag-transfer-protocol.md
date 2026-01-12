---
description: 'Creator: Jun Wei'
---

# Flag Transfer Protocol

## Description

{% code overflow="wrap" %}
```
What do you know about File Transfer Protocol (FTP)? I've heard many people call it the Flag Transfer Protocol!
```
{% endcode %}

Category: Forensics (Easy)

## Walkthrough

Nothing much to be said, opening the network capture immediately gives us the flag in plain text

<figure><img src="../../.gitbook/assets/image (227).png" alt=""><figcaption></figcaption></figure>

## Conclusion

A friendly reminder to avoid using plain FTP as all data transferred is plaintext, including your username and password, as well as any confidential files that you're downloading/uploading. Instead, use secure alternatives, such as FTPS (FTP-SSL or FTP Secure)
