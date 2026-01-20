---
description: 'Category: OSINT (Easy)'
---

# Traveling Intern 3

## Description

```
Refer to the assignment brief for details and objectives

Flag format is YBN{location_name_all_lowercase_no_spaces}

E.g. if the location was Changi Airport, the flag would be YBN{changi_airport}
```

Assignment Brief:

{% code overflow="wrap" %}
```
Good job, intern. We were able to deploy agents to the new meetup point, and they got a hint on where the next location will be posted. One of the agents mentioned that they overheard William saying to “check other socials” for the next location meetup, and that they will be getting the intern to post the new location.

Find the platform that the intern will be posting the next meetup location, and where the meetup will be organized. We suggest using automated tools for this assignment. 
```
{% endcode %}

## Walkthrough

As the assignment brief suggested, we can use automated tools to make life easier when finding the other socials that the intern has posted the next location on. I'll be demonstrating two tools, [Sherlock](https://github.com/sherlock-project/sherlock) and [Blackbird](https://github.com/p1ngul1n0/blackbird).&#x20;

<figure><img src="../../../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (284).png" alt=""><figcaption></figcaption></figure>

Both tools correctly identify the Pastebin account, which contains one note. However, this note is password-protected, which means we have to provide a valid password before we can see it

<figure><img src="../../../.gitbook/assets/image (286).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (287).png" alt=""><figcaption></figcaption></figure>

If we look at the intern's Instagram account, we can see a suspicious string in the profile, which looks kind of like a password.

<figure><img src="../../../.gitbook/assets/image (288).png" alt=""><figcaption></figcaption></figure>

We can use this to unlock the bin, and read the contents.

<figure><img src="../../../.gitbook/assets/image (289).png" alt=""><figcaption></figcaption></figure>

This string is an Uber H3 cell ID, which I've covered before in a previous CTF writeup [here](https://blog.northland.dev/writeups/2024/csit-tisc/level-1). We can input the ID into [https://h3geo.org](https://h3geo.org), and get the location associated with this cell.

<figure><img src="../../../.gitbook/assets/image (290).png" alt=""><figcaption></figcaption></figure>

Looks like the final location, probably when the group is flying home from Shanghai. Lets submit this answer and get our flag.

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

Honestly this one made me laugh the hardest. Participants were able to somehow find the answer for Part 3 first, before Part 2.&#x20;

Apart from that slight hiccup, there honestly isn't much to talk about. I did learn about Blackbird, which seems to be a much better OSINT tool than Sherlock, although it may have a smaller list of platforms where the tool searches.&#x20;
