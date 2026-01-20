---
description: 'Solved by: Ryo Soikutsu'
---

# Order of Pixels

## Challenge Description

{% code overflow="wrap" %}
```
You stumble upon a mysterious folder, filled with seemingly random images. At first glance, they look ordinary, but something feels off. The order of the pixels holds a secret, and its up to you to find the secret.
```
{% endcode %}

Category: Steganography (Easy)

## Walkthrough

Extracting the 7zip file, we are provided with a folder filled with images. Looking through some of the files extracted, it seems like all of them are blank.&#x20;

<figure><img src="../../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

Checking the extracted file with the ls command, it seems like there's 4 image files which are suspiciously large

<figure><img src="../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

Hmm, looks suspicious. We can confirm our suspicions using the [StegExpose](https://github.com/b3dk7/StegExpose) tool.&#x20;

<figure><img src="../../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

We can copy these 4 files out to a separate folder, and attempt further analysis on them. We can throw the 4 files into Aperi'Solve to see if anything comes up

<figure><img src="../../.gitbook/assets/image (119).png" alt=""><figcaption><p>Suspiciously flag-shaped picture</p></figcaption></figure>

Looks very suspicious, lets throw the other 3 files in

<figure><img src="../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

We can reconstruct the flag using the different fragments in the image.&#x20;

## Conclusion

This challenge took quite long to solve, largely because I assumed that all of the images provided will be involved in one way or another. I tried several other techniques, such as LSB steganography and analyzing the hex data directly, to no avail. Once again, overthinking a simple challenge led to me wasting time and energy finding a solution.
