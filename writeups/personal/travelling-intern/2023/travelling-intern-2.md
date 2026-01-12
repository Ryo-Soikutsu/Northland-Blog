---
description: 'Created by: Ryo Soikutsu'
---

# Travelling Intern 2

## Description

{% code overflow="wrap" %}
```
While we're dealing with the aftermath of the cyberattack, one of our lead investigators has taken the liberty to go on a vacation without informing anyone. We need to track down their whereabouts and ensure they are safe. Can you help us find them?

Flag format: NYP{location_they_are_at} (lowercase separated by underscore)
```
{% endcode %}

Category: OSINT

(NOTE: This is a recreation of the original challenge. The original challenge was created in 2023, whereas the recreated challenge was created in 2025)

## Walkthrough

Downloading the provided file, we are provided with a HEIC (High-Efficiency Image Format) file, which is the default image format for Apple devices. When we open the image, we see the following

<figure><img src="../../../.gitbook/assets/image (91).png" alt=""><figcaption><p>Very beautiful view out of a pavilion</p></figcaption></figure>

Investigating the image further, we can see that it contains some metadata of the device that it was taken on, as well as location information when the image was taken

<figure><img src="../../../.gitbook/assets/image (92).png" alt=""><figcaption><p>Viewed on Windows with the Properties tab</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (83).png" alt=""><figcaption><p>Viewed on Linux using the exiftool command</p></figcaption></figure>

With this information, we can convert the latitude and longitude from DMS format to decimal degrees, and search it in Google Maps

<figure><img src="../../../.gitbook/assets/image (84).png" alt=""><figcaption><p><a href="https://www.fcc.gov/media/radio/dms-decimal">https://www.fcc.gov/media/radio/dms-decimal</a></p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

There are two shops close to the provided coordinate. We can try both of them to see which is the correct answer. We find that the craft shop is the correct answer

<figure><img src="../../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

This was one of the first challenges that I had made, part of the Travelling Intern series (first released in YBN CTF 1.0 in 2023).&#x20;

One of the common pitfalls encountered by participants is that they pass the image directly into Google Lens, in an attempt to find the location. Unfortunately, Google Lens will return the incorrect location, as shown below

<figure><img src="../../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

The matches found varies every time you search. However, the closest result returned so far is Otowa Saryo (found in YBN CTF 1.0), which is right next to the craft shop

<figure><img src="../../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

Other participants have also resorted to the Geoguesser approach, looking for the location manually using various landmarks such as the houses and background scenery shown in the image. However, they are generally far off from the correct location, about 50-100m away (the red circle area indicates the general location guessed by participants)

<figure><img src="../../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

