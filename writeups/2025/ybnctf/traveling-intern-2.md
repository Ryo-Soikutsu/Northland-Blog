---
description: 'Category: OSINT (Easy)'
---

# Traveling Intern 2

## Description

{% code overflow="wrap" %}
```
Refer to the assignment brief for details and objectives Flag format is YBN{location_name_all_lowercase_no_spaces}

E.g. if the location is Resorts World Sentosa, the flag will be YBN{resorts_world_sentosa}
```
{% endcode %}

Assignment Brief:

{% code overflow="wrap" %}
```
Good job on finding William Kronx’s hotel, we have deployed agents to gather field intelligence on the target and his companions. Unfortunately, it seems like William has caught wind of ICA movement and has started to obfuscate his movements and meetups to prevent agents from keeping up with him. 

We noticed that William has made a comment on one of the intern’s social media posts but deleted it before we were able to read it. Recover the original comment that William has made, maybe it contains some hints on the next meetup with his… business associates.
```
{% endcode %}

## Walkthrough

Based on the hints provided by the assignment brief, it seems that we're being tasked to recover a deleted comment on one of the Instagram posts on the intern's account. First, we have to identify which post was the one where the comment was made. Looking through the posts, we identify this [one](https://www.instagram.com/p/DSEJlHOEzTe/?igsh=NzVwbmprODV3Ymp0), which has comments enabled.&#x20;

<figure><img src="../../.gitbook/assets/image (22).png" alt="" width="375"><figcaption></figcaption></figure>

Of course, the comment is nowhere to be seen. However, we can use archive sites such as Wayback Machine to turn back time and see an older version of the site (provided it has been archived before). Unfortunately, Wayback Machine itself has an unusual quirk, where it cannot parse dynamic and content-heavy websites such as Instagram, so we'll have to use a different site.&#x20;

Another archive site is [https://archive.is/](https://archive.is/), which does support archiving Instagram. We can copy the URL of the suspect post, and paste it into the site to search for hits.&#x20;

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Remember to remove the tracking tags at the end, to avoid messing up the search. The archive site returns the following [webpage](https://archive.is/7HD2E), which does indeed contain a comment. The comment has a Base64-encoded string, which we can decode easily to get a set of coordinates.

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Looks like the group travelled to Shanghai Disneyland, where the next meetup is at. Man, the intern sure is lucky to have such a kind and caring boss...



<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

At this point, you may have noticed that the flag format hints at the kind of location the challenge is asking for.&#x20;

This challenge caused some participants to have some issues, because they somehow discovered the answer for Part 3 instead of Part 2. Kind of funny, will keep in mind to not release all the challenge resources at one go in future.
