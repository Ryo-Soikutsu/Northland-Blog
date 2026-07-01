---
description: 'Category: Forensics (Hard)'
---

# バカ通信 (Stupid Communication)

## Description

{% code overflow="wrap" %}
```
Suspect left his device and we managed to get a disk image before he returned. We believe this is just one of many devices he owns and a preliminary investigation has determined he likes to use Github, but this disk does not have what we really need. Smart enough not to store Github credentials as a plain txt file, but it has to be somewhere on this device...

Get dist idiot_communication.E01 from https://storage.googleapis.com/sctf/idiot_communication.E01

Note that I removed a lot of system files to debloat, open a ticket if you have any issues
```
{% endcode %}

## Walkthrough

We're provided with an E01 disk image file of a Windows system, which we can load into FTK Imager for further analysis.

<figure><img src="../../.gitbook/assets/image (435).png" alt=""><figcaption></figcaption></figure>

Based on the description provided, we can assume that we're looking for Git credentials, or something that can give us access to a private GitHub account. Looking inside John Rich's Documents folder, we find a repo cloned here named NOTTheFlag

<figure><img src="../../.gitbook/assets/image (436).png" alt=""><figcaption></figcaption></figure>

As implied, the contents of this repo doesnt contain the flag we're looking for. We can try to check out the .git folder in case there's any SSH keys or credentials stored here, but we dont find anything either

<figure><img src="../../.gitbook/assets/image (437).png" alt=""><figcaption></figcaption></figure>

Looking in the user's AppData folder, we notice a directory for Signal, a communications app primarily focused around privacy.&#x20;

<figure><img src="../../.gitbook/assets/image (438).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (439).png" alt=""><figcaption></figcaption></figure>

### Decrypting Signal

> Note: everything in this section is based off this blogpost linked [here](https://stolenfootball.github.io/posts/research/2025/signal_windows_decryption/)

Signal stores its core database in sql/db.sqlite, a password-protected database with a randomly-generated password. The password is typically dozens of characters long, and is infeasible to brute-force it manually. Instead, we can recover the password used using the following files:

* Roaming/Signal/Local State - A DPAPI blob containing an encrypted key
* Roaming/Signal/config.json - The database key encrypted with AES-GCM
* Roaming/Microsoft/Protect/\<SID>/\* - All past and present DPAPI master key blobs

We also need the SAM, SYSTEM and SECURITY registry hives, stored in Windows/System32/config. These three hives will be necessary to recover the user's password for the DPAPI decryption process. Once all 6 files (as well as the encrypted database) are exported and copied out, we can start the decryption.

First, we need to obtain the correct masterkey for this system. We can make use of a tool called [Pypykatz](https://github.com/skelsec/pypykatz) to perform most of the decryption for us (supposedly, Impacket's [dpapi.py](https://github.com/fortra/impacket/blob/master/examples/dpapi.py) works too, but it failed for me).&#x20;

First, we extract the credentials for the user using Impacket's secretsdump.py script

<figure><img src="../../.gitbook/assets/image (440).png" alt=""><figcaption></figcaption></figure>

We can copy the NT hash portion, and use CrackStation to attempt to find a matching plaintext password, which it successfully does.

<figure><img src="../../.gitbook/assets/image (441).png" alt=""><figcaption></figcaption></figure>

Next, we generate a set of prekeys, which we can use to derive the master key later.

<figure><img src="../../.gitbook/assets/image (442).png" alt=""><figcaption></figcaption></figure>

Then, we can attempt to calculate the correct master key. During the solving of this challenge, I couldn't remember how to identify the right key, so I simply generated keys for all 3.

<figure><img src="../../.gitbook/assets/image (443).png" alt=""><figcaption></figcaption></figure>

Finally, we convert the DPAPI blob stored in Local State to hex format, and attempt to decrypt it using the three supposed masterkeys.

<figure><img src="../../.gitbook/assets/image (444).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (445).png" alt=""><figcaption></figcaption></figure>

With the first key obtained, we can move on to decrypt the second key, and access the database.&#x20;

According to the blogpost, Signal stores the variables needed to decrypt the database's key in config.json, in the following format

```json
{
    "header": "BYTES [1-3]",
    "nonce": "BYTES [4-16]", // aka the Initialization Vector (IV)
    "encrypted_data": "BYTES [17-81]", // This contains the actual key itself
    "mac": "BYTES [82-98]" // aka the GCM Tag
}
```

For this example, the fields will appear as such

{% code overflow="wrap" %}
```json
{
    "header": "763130",
    "nonce": "0bccaaeb808648ac871b3fbd",
    "encrypted_data": "06e828489c5d6cbe5f16048d00b775a3779ab79c8197feb50ae5b8c7f0b50aea98422dcf735d1aaba8de5135825786a32dd9201d4c7d11e935d9cd14e69541a3",
    "mac": "acbace3a99bddbe3cb48424f3b0bfbcf"
}
```
{% endcode %}

We can use this CyberChef recipe to decrypt the database key. Take note that the key specified in this recipe refers to the key we obtained earlier via DPAPI decryption

<figure><img src="../../.gitbook/assets/image (446).png" alt=""><figcaption></figcaption></figure>

Finally, we can use this key to decrypt the database. Open the DB in sqlitebrowser, and select "Raw key" in the password field. Paste in the decrypted key in the format 0x\<key>, and click enter.

<figure><img src="../../.gitbook/assets/image (447).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (448).png" alt=""><figcaption></figcaption></figure>

Individual messages in Signal are stored in the aptly-named messages table. Browsing the data for this table yields us a GitHub token, as well as a repository.

<figure><img src="../../.gitbook/assets/image (449).png" alt=""><figcaption></figcaption></figure>

### Enumerating GitHub (not really)

We now have a GitHub personal access token (PAT), which "are an alternative to using passwords for authentication to GitHub when using the [GitHub API](https://docs.github.com/en/rest/overview/authenticating-to-the-rest-api) or the [command line](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#using-a-personal-access-token-on-the-command-line)" according to GitHub's own [documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens). Notably, this means that we can use the PAT to query GitHub's API as the user it was generated as, and enumerate their profile. To start, we will attempt to query the /user endpoint, to test which user we can request information as. Use the following command to perform the query

{% code overflow="wrap" %}
```bash
curl -H "Authorization: Bearer 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" https://api.github.com/<api-endpoint>
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (450).png" alt=""><figcaption></figcaption></figure>

We're successfully authenticated as JohnRichFromSCTF, which we've already established is the user we're hunting for. We can enumerate the repos that are available as such

<figure><img src="../../.gitbook/assets/image (451).png" alt=""><figcaption></figcaption></figure>

This gives us a lot of detailed information about each repo that this token can access, such as description, URLs and more. Notably, we see a repository called idiot-communication (the same name as this challenge), and our flag there in plaintext.

<figure><img src="../../.gitbook/assets/image (452).png" alt=""><figcaption></figcaption></figure>

## Conclusion

