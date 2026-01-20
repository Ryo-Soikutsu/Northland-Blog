---
description: 'Category: Forensics (Hard)'
---

# Secure The Network v2

## Description

{% code overflow="wrap" %}
```
Following a previous cyberattack on the organization's website, the IT team has set up a brand new webserver, and performed a penetration test on it to evaluate its vulnerabilities. SOC has been provided with the logs generated from the practice, and has requested you to investigate it and identify any vulnerabilities in the new system.
```
{% endcode %}

## Walkthrough

We're provided with two files, a network capture (<mark style="color:red;">**attack\_linux\_1.pcap**</mark>) and a tar zip file (<mark style="color:red;">**catscale.tar.gz**</mark>). We'll start with the network capture, and see what we are working with.

### Network Analysis

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

That is a lot of packets, lets try to filter it by HTTP packets to look for potential attacks

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

There are a lot of attempts to access seemingly-random URLs, which might indicate a potential brute-force attack. We can confirm this by selecting on a packet, right clicking and select "Follow" -> "HTTP Stream".&#x20;

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

The user agent tells us that the attacker is using Gobuster to brute-force valid directories on the webserver.&#x20;

<details>

<summary>What is Gobuster?</summary>

**Gobuster** is a powerful open-source tool designed for directory and DNS brute-forcing, widely used in ethical hacking and penetration testing. By leveraging wordlists and customizable options, Gobuster empowers penetration testers and bug bounty hunters to map out the attack surface of a target system. It can reveal critical assets that may otherwise remain undetected, providing valuable insights during security assessments. Unlike automated crawlers, Gobuster focuses on brute-forcing, making it a reliable solution for controlled and precise testing.

* Source: [https://gobuster.org/what-is-gobuster-and-how-does-it-work/](https://gobuster.org/what-is-gobuster-and-how-does-it-work/)

</details>

Gobuster generated a lot of noise, so we'll just scroll down until we find more non-bruteforce traffic.&#x20;

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

The webserver seems to be running Wordpress, which commonly has vulnerabilities from the plugins that are installed on the website. Further down, there seems to be more enumeration, although there aren't any obvious hints to indicate which tool is being used. After the enumeration, there seems to be suspiciously specific requests to a certain directory

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

The attacker is attempting to access the README file for the Wordpress File Manager plugin, possibly to get the version of the plugin and find a CVE.&#x20;

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

The plugin is running on version 6.0, as you can see in the changelog. If we search up "wordpress file manager 6.0 cve", we get a couple of results

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

To confirm that this CVE is being exploited, we can check the later requests

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

These requests to <mark style="color:$danger;">**/wp-content/plugins/wp-file-manager/lib/php/connector.minimal.php**</mark> does confirm that the attacker exploited CVE-2020-25213, an unauthenticated arbitrary file upload vulnerability that leads to remote code execution (RCE), likely using the Metasploit payload ([here](https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/multi/http/wp_file_manager_rce.rb)).&#x20;

If we do a filter by ICMP (ping) requests, we can see some unusual requests being made in sequential order.

<figure><img src="../../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>

The destination IP address increasing in sequential order strongly indicates that the attacker may have started host enumeration for lateral movement, after gaining initial foothold on the webserver. To see what hosts were discovered by the attacker, we can set the following filter.

<figure><img src="../../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>

We can guess that the .254 IP address is a gateway, since gateway addresses tend to be set to .254 or .1. This leaves us with .125, which we can also infer is likely a Windows system, due to the TTL (Time To Live) value of 128.

### Catscale Log Analysis

Unzipping the tar archive, we're provided with a couple of folders.

<figure><img src="../../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

Lets start by checking the user files, to see if there are any interesting files that the attacker may have dropped or exfiltrated.

<figure><img src="../../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (276).png" alt=""><figcaption></figcaption></figure>

The root user's bash history does not have much of note, although it does confirm what we originally expected about the plugin name and version. The user ravin's history is much more interesting, as we can see the attacker gaining persistence and dropping additional post-exploitation tools for privilege escalation and lateral movement

<details>

<summary>What is LinPEAS?</summary>

LinPEAS is an open-source script designed to automate the process of searching for potential privilege escalation vulnerabilities on Linux/Unix/macOS systems. It is a tool developed by [**Carlos Polop**](https://github.com/carlospolop), a cybersecurity researcher and penetration tester, and is widely used by system administrators and security auditors to identify and address vulnerabilities before they can be exploited by attackers.

It does this by scanning the host system for a massive range of potential vulnerabilities, including but not limited to

* Misconfigured sudo permissions
* Exploitable kernel versions
* Vulnerable SUID/SGID binaries
* Writable files in sensitive directories
* Misconfigured services

</details>

Just to be thorough, lets check the processes that Catscale has detected on the system. In <mark style="color:red;">**webserver-20251013-1008-process-cmdline.txt**</mark>, we see a suspicious command being executed.

<figure><img src="../../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>

This command is part of a series of commands, commonly used to upgrade from an initial webshell an interactive TTY shell. It is also referred to as "stabilizing a shell".&#x20;

<details>

<summary>Trivia: The full set of commands</summary>

Here is the basic (and common) set of commands that is used to stabilize a shell

```bash
python3 -c "import pty;pty.spawn('/bin/bash')"
# Ctrl + Z to background the shell and return to your attacker terminal
stty raw -echo;fg
export TERM=xterm-256color
```

Sometimes, you may choose to run additional commands, to add further customizability or stabilize the shell further. Some of these commands are listed below

```bash
export SHELL=bash
stty rows 40 columns 100
export PS1="\u@\h:\w\$"
```

</details>

The command seems to be partially corrupted, likely due to the way that logs are gathered. However, we can easily fix this since we know the original command, and can thus infer the missing characters.&#x20;

### Answers

Now that we've concluded the forensics investigation, we can now connect to the server and get our flag.

1. What is the IP address of the attacker machine? - 120.225.134.131
2. What application is the Linux webserver running? Answer in all lower-case - wordpress
3. What is the vulnerable WordPress plugin that the attacker identified? Format: wordpress\_plugin\_in\_lowercase:version\_number - file\_manager:v6.0
4. What CVE did the attacker use to exploit the webserver? Format: CVE-XXXX-XXXX - CVE-2020-25213
5. What was the first command that the attacker ran after gaining initial access using the CVE? Answer is case-sensitive - python3 -c "import pty;pty.spawn('/bin/bash')"
6. The attacker uploaded some files onto the webserver for privilege escalation and further enumeration. What are the files uploaded? - host\_discovery.sh,linpeas.sh,port\_discovery.sh
7. The attacker dropped another file on the webserver for persistence. Provide the filename of this file - id\_ed25519.pub
8. The attacker attempted to enumerate hosts on the internal network, and discovered one exposed host. What is the IP address of the host? - 172.16.100.125

<figure><img src="../../.gitbook/assets/image (280).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

Secure The Network 2 was originally meant to be a 2-part investigation: the initial access to the internal network via the webserver, and then the actual domain compromise. However, I ran into a myriad of issues during the creation of the second part. I have thus pushed the second part to Secure The Network 3.

Moving onto the actual attempts made by participants, it seems like many of them were able to correctly guess that for Q5, you have to fix the double quotes before submitting. However, many were also not able to get the answer for Q4, due to the additional "v" that had to be added to the answer. This slight issue is intentional, but I will keep this in mind for future challenges.

Overall, I had a lot of fun with creating this challenge. It was a fun practice with making multi-network investigations, and I hope participants had fun with solving this. Due to the rising issue with players using OpenAI's Codex AI to solve challenges, I have also started looking into potential counters to deter AI players, but will likely switch paths to cater towards human players instead, hopefully to encourage them to solve manually.&#x20;

Edit: An interesting observation that I had made about the winners that were interviewed, they all seemed to handicap themselves by only checking the network capture file, and ignored the additional Catscale investigation results. As such, a few of these players encountered issues when attempting to mentally build a timeline of the attack, since information such as missing commands were not present in the network capture.&#x20;
