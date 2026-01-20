---
description: 'Creator: wrenches'
---

# sharktel

## Description

```
help log into my router :(
```

Category: Forensics (Easy)

## Walkthrough

We are provided with a network capture, which we can open in Wireshark

<figure><img src="../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

Straight away, we notice a HTTP stream in the capture, which could contain credentials or other interesting information. Right click -> Follow -> HTTP Stream to follow the specified packet

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

Looking at the password field, it looks like the user is trying to use SQL injection to enumerate the password of the specified user. We can copy out the command and URL decode it to see the actual payload being used

<figure><img src="../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

The attacker is using the substr command to enumerate each character of the password. When the server responds with a 200 OK response, the attacker will move on to the next character in sequence, and continue until the full password is recovered. We can filter for successful HTTP responses using Wireshark's packet filters feature, as such

<figure><img src="../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

Now we can look through each one, and extract each character one by one. Remove any duplicate commands to avoid being confused.&#x20;

<figure><img src="../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

With valid credentials, we can login to the router website and get our flag

<figure><img src="../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

## Conclusion

This was a bit of a tedious challenge, having to copy and paste the command out from Wireshark to CyberChef to decode. There is likely a command that you can use to make this easier, but I didnt bother to figure it out. The short read about the author's past was heartwarming and interesting to read, about how they got into networking and cybersecurity. It's always nice to see little tidbits or easter eggs left in challenges for the participant to discover.
