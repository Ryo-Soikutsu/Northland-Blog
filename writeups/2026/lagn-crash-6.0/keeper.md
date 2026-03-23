---
description: 'Category: Forensics (Medium)'
---

# Keeper

## Description

{% code overflow="wrap" %}
```
I found this database file in one of our server backups, but I lost the password to it. Can you help me recover it?

GDrive: https://drive.google.com/file/d/1m1koO9QI2FVi8jbyfpu47H4EYRbLm9-d/view?usp=drive_link
```
{% endcode %}

## Walkthrough

We are provided with two files, a KeePass database file (flag.kdbx) and a memory dump (KeePass.DMP). We can open the database in a KeePass client, which will prompt us for the credentials to decrypt the contents.

<figure><img src="../../.gitbook/assets/image (347).png" alt=""><figcaption></figcaption></figure>

We can try all kinds of passwords, and there is a high chance we will not be able to unlock it. Instead, we can turn to the memory dump and attempt to extract credentials from it. By doing a Google search for KeePass vulnerabilities, we come across an interesting article.

<figure><img src="../../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>

The article talks about CVE-2023-32784, a vulnerability in the way KeePass handles password fields and erases sensitive information from memory. In short, KeePass uses a custom textbox for password entry, which will leave leftover strings in memory each time a character is entered in the field. These strings can be extracted by an attacker either from a process dump, page swaps, hibernation files or a full RAM dump from the computer. For example, the leftover strings for the password "P@ssw0rd123" will appear like this:

```
•@
••s
•••s
••••w
•••••0
••••••r
•••••••d
••••••••1
•••••••••2
••••••••••3
```

Due to the nature of the exploit, the first character cannot be extracted. However, it can be brute forced very easily offline.

Several POCs are available online for this CVE, so we'll use z-jxy's KeePass dumper Python script (found [here](https://github.com/z-jxy/keepass_dump)). We can run this against the memory dump provided to extract a partial password

<figure><img src="../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>

Just like it was advertised, the first password could not be extracted. Since the password appears to be a dictionary phrase, we can safely assume the first character is either "S" or "5" (to form the word "secure"). We can try both options to see which one is the correct password.

<figure><img src="../../.gitbook/assets/image (350).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (351).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (364).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

:)

(If you know what the other set of credentials is for, keep it to yourself\~)
