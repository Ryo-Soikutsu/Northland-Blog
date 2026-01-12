---
description: 'Creator: Ryo Soikutsu'
---

# Secure The Network v1

## Description

{% code overflow="wrap" %}
```html
After deployment of the Vault, we received a report of a critical vulnerability in it. Unfortunately, we were too late to fix it, and a cyberattack was successfully launched against it. We were able to recover the system logs from our firewall and web server, as well as a capture of the network traffic at the time of the attack. Can you take a look at these files, and find anything interesting?
```
{% endcode %}

Category: Forensics

## Walkthrough

### Question 1 - What is the IP address of the attacker?

There are a couple of ways to look for the following value, but this walkthrough will showcase using the Apache2's `access.log` log file

<figure><img src="../../.gitbook/assets/image (172).png" alt=""><figcaption></figcaption></figure>

We can see that the IP address 201.172.32.43 shows up very often, indicating that they may be performing directory enumeration on the web server. We can dump out all the IP addresses present in the log files using the following command

<figure><img src="../../.gitbook/assets/image (173).png" alt=""><figcaption></figcaption></figure>

Now we see that 201.172.32.17 has much higher occurrences, maybe that's the attacker's IP address? Now we can use the network capture file provided to figure out which IP address is the correct one

<figure><img src="../../.gitbook/assets/image (174).png" alt=""><figcaption></figcaption></figure>

Filtering packets from 201.172.32.17, we can see that most of the traffic generated is HTTP traffic, which suggests that this machine is the attacker's enumeration machine. Could be the correct answer, so let's take note of that

<figure><img src="../../.gitbook/assets/image (175).png" alt=""><figcaption></figcaption></figure>

Filtering for packets from 201.172.32.43, we can see that this machine performs some SSH connections to a private server, as well as some signs of directory traversal in the HTTP traffic. This is the attacker's machine, as it performs the directory traversal attack to expose the SSH key on the server, and then uses it for initial access

{% hint style="info" %}
For readers who are curious as to what 201.172.32.17 is, it was intended to be a red herring to trick CTF players, and to make sure that they double check their answers before submitting. :P
{% endhint %}

### Question 2 - What is the public IP address of the target?

To protect the target, NAT (network address translation) has been configured on the firewall. Since the firewall used is PFSense, we can access the NAT information in the provided config backup at `export-syslog.zip/config-pfsense.ybn.org.xml`

<figure><img src="../../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

### Question 3 - What tool did the attacker use to scan the target? Answer in all lower-case

Answer can be found in Q1, in `access.log` under the request header field

### Question 4 - What is the vulnerability used by the attacker on the target? Answer in all lower-case

There are several possible answers to this question. Searching up the indicators of compromise on Google (or ChatGPT), we get&#x20;

<figure><img src="../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

We can try each of the answers to find the accepted answer

### Question 5 - Give the timestamp when the enumeration process began. Answer in UTC (Format is MMM:DD, YYYY HH:MM:SS)

The enumeration that the question refers to is the start of the attacker enumerating the target server. We can check the `access.log` file to find the timestamp of the first enumeration attempt

<figure><img src="../../.gitbook/assets/image (177).png" alt=""><figcaption></figcaption></figure>

### Question 6 - What is the first directory enumeration tool that the attacker uses? Answer in all lower-case

Common directory enumeration tools are Gobuster, Dirbuster, Ffuf and Feroxbuster. Knowing this, we can search for these tool names in `access.log`, or alternatively look through the logs manually until we find one of the listed tools in the request header

<figure><img src="../../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

### Question 7 - The attacker attempted to access the web server through SSH. What is the timestamp for the successful access? Answer in UTC (Format is YYYY-MM-DD HH:MM:SS)

All logon attempts, be it SSH or console/GUI, to a Unix system is stored in the `/var/log` folder (either `auth.log` for Debian or `secure` for RHEL)

We can search for all occurrences of SSH login attempts, then filter it by the attacker's IP address, and then by the success status

<figure><img src="../../.gitbook/assets/image (179).png" alt=""><figcaption></figcaption></figure>

### Question 8 - The attacker attempted to erase any traces of their attack by deleting system logs. Which files were modified by the attacker Answer in filename 1\_filename 2

This is a trickier challenge. Usually, attackers will attempt to erase any authentication attempts or traces of their exploit methods from system logs. They may try to wipe `access.log`, `error.log`, `auth.log`/`secure` and/or `syslog`/`messages`

* Note: `syslog`/`messages` is the logfile for the system's syslog module, which stores the log files generated by applications running on the machine

Searching through these 3 log files, we can see these notable chunks of timeskip

<figure><img src="../../.gitbook/assets/image (180).png" alt=""><figcaption><p>apache2/error.log</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (181).png" alt=""><figcaption><p>auth.log</p></figcaption></figure>

### Question 9 - Upon further analysis of our firewall logs, it seems like one of the sysadmins had left in a rule which allowed for the attacker to access the remote server through&#xD; SSH. Identify this overly-permissive rule. Answer is the rule's tracker ID

This question is asking for a PFSense firewall rule ID. The exported firewall configuration can be found in the `export-syslog.zip` file, named `config-pfsense.ybn.org.xml`

<figure><img src="../../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

Since the question specifically asks us to look for a rule allowing SSH access, we can search for that specific term

<figure><img src="../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

Once we know what entry our target rule is, we can scroll up until we find the tracker ID

## Creator's Notes

This was my first incident response forensics challenge, so it was slightly scuffed. There were a couple of hiccups during the log recording process, which showed up in the logs. Most notably was the initial access process, when I attempted to access the SSH private keys using directory traversal. In the end, I just added rwx permissions to the key, and continued the attack from there.&#x20;

Since it was a different challenge format than the usual flag format, I had to create a custom trivia server for the player to submit their answers to. The following scripts serve the following

```python
import os
import json

BANNER = "Logged into the Northland Forensics server. Answer all questions to complete the assignment.\n"

with open("questions.json") as f:
    questions: list = json.load(f)["questions"]

print(BANNER)
for i, question in enumerate(questions):
    print(f"Question {i+1}: {question['question']}\n")
    answer = input("> ")
    if answer != question["answer"]:
        print("Incorrect answer. Exiting...")
        exit(1)

print("All questions answered correctly. Assignment marked as completed.")
print(f"Admin message: {os.getenv("FLAG")}")
```

```docker
FROM python:3.12-alpine

WORKDIR /usr/src/securethenetwork

RUN apt-get update && apt-get install -y socat && apt-get clean && rm -rf /var/lib/apt/lists/*

COPY . .

RUN addgroup -S physgrp && adduser -S physuser -G physgrp
RUN chown -R physuser:physgrp /usr/src/physical
USER physuser

EXPOSE 1337

ENV FLAG=YBN{FAKE_FLAG}

CMD ["socat", "-dd", "TCP-LISTEN:1337,fork,reuseaddr", "EXEC:python3 server.py"]

```

The initial trivia server works fine, however there was feedback that the answers were not clear at time, especially with Q4. Future iterations of the trivia server will include a Hangman-style hint, which indicates the length of the answer.&#x20;

## Final Notes

The Github repo link for this particular challenge can be found [here](https://github.com/Ryo-Soikutsu/Secure-The-Network/tree/main/STN-v1)
