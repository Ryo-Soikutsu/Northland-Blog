# What are CTFs

You must have came from the previous page, wondering what a CTF is.&#x20;

Well, let me give you a brief (and hopefully comprehensive) rundown of them for you.

CTFs (also known as capture-the-flag) are a special kind of competition, mainly aimed at cybersecurity. There are two primary forms of CTFs: Jeopardy style and Attack-Defense. For the scope of this blog, we will only cover Jeopardy style. (Maybe I'll cover attack-defense in future, when I learn how to play them)

In Jeopardy-style CTFs, participants are given a list of challenges, and are required to obtain a flag to complete the challenge.&#x20;

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption><p>Example of a challenge in a CTF. Sanity checks are very common to introduce new players to the mechanics of CTFs</p></figcaption></figure>

Challenges are typically split into 7 different categories:

* Cryptography: covers various forms of cryptography, such as ciphers, encryption and encodings.
* Forensics: covers primarily steganography, but sometimes will include disk forensics and network log analysis .
* Web: covers web exploitation. (e.g. SQL injection, directory traversal)
* Binary exploitation ("pwn"): covers binary exploitation, as the name suggests. Usually involves some form of reverse engineering.
* Reverse engineering ("rev"): covers reverse engineering. Unlike pwn, reverse engineering is not limited to binaries, but can also include Python, WASM (web assembly) and Javascript, to name a few.
* Open Source Intelligence ("OSINT"): legal stalking basically :P Usually requires the use of social media (e.g. Instagram, Facebook) and some Googling skills to find some information about the target.
* Miscellaneous ("misc."): a catch-all for challenges that do not fall under the scope of the previous 6 categories. Programming challenges are sometimes categorized under misc., if they do not get their own category.

Furthermore, they are also sorted onto three (main) difficulties:

* Easy: Beginner-friendly challenges, usually requires only surface-level knowledge of the category (e.g. Inspect Element on a webpage to get the flag). Some challenges may require that participants know of common vulnerabilities. (e.g. SQL injection with no input validation)
* Medium: Harder challenges, may require that participants have actual knowledge of how a vulnerability works, instead of just copying a solution online. Some may have multiple layers of vulnerabilities to go through. (e.g. JWT token manipulation to bypass authentication, then directory traversal to access the flag)
* Hard: Usually the highest level of difficulty, will require that participants have an excellent grasp of a range of vulnerabilities or actual CVEs to complete the challenge. May also require that participants have an in-depth understanding of how the underlying systems work. (e.g. the underlying architecture of a processor)
* Insane: Relatively uncommon category, since most challenges can be classified under Hard. If you are trying to solve Insane challenges, you are either really insane, or actually have skill at this. Either way, this blog won't be of much help for them, so good luck!&#x20;

There are a plethora of online tools and resources available for participants. Here are a few of my favorites:

* [SecLists](https://github.com/danielmiessler/SecLists): A compilation of wordlists for a range of purposes. Extremely useful for web exploitation, such as directory fuzzing and brute-forcing logins.
* [SQLi Payload List](https://github.com/payloadbox/sql-injection-payload-list): Similar to SecLists, contains a large range of SQL injection (SQLi) commands and queries.
* [CyberChef ](https://gchq.github.io/CyberChef/)and [dCode](https://www.dcode.fr/en): Very useful tools for forensics and cryptography.
* Google: Your best friend. Googling is often allowed, and even encouraged, for CTFs. Writeups for the challenges may also already exist, and you can learn from them. (Please do not copy-paste the solution, it defeats the purpose of doing them)
