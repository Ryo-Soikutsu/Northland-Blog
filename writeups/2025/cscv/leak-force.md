---
description: 'Solved by: Ryo Soikutsu'
---

# Leak Force

## Description

```
<NO DESCRIPTION>
```

Category: Web

## Walkthrough

Navigating to the website, we are presented with the following webpage

<figure><img src="../../.gitbook/assets/image (240).png" alt=""><figcaption></figcaption></figure>

As the website title suggests, this website has an IDOR vulnerability. We can load the page up in Burpsuite browser and poke around the site to examine the network traffic generated

<figure><img src="../../.gitbook/assets/image (241).png" alt=""><figcaption></figcaption></figure>

This is pretty suspicious, the server queries the API for the profile information of the current user, and uses a parameter "id" to indicate what user to get data for. We can intercept the traffic from the client to the server, and change the user ID of the profile being queried to another user's number

<figure><img src="../../.gitbook/assets/image (242).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (243).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (245).png" alt=""><figcaption></figcaption></figure>

As expected, we get the profile information of the admin user. However, this doesn't mean that we've taken control of the admin user's account. To do so, we can try to reset the admin's password below, and change the user ID to the admin's ID

<figure><img src="../../.gitbook/assets/image (246).png" alt=""><figcaption></figcaption></figure>

Now we can log back into the admin account using our new credentials, and get the flag

<figure><img src="../../.gitbook/assets/image (247).png" alt=""><figcaption></figcaption></figure>

As you can see, there are artifacts of other participants' attempts to gain access to the flag.
