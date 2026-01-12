# Fixing issues with the network

## Introduction

After testing out the network and completing the Linux portion of the attacks, I discovered a whole slew of issues with the Active Directory network that made it annoying to do the attacks. For instance, the workstations would not have the necessary configurations to allow RDP access for any user, no matter what groups they were in

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

Or systems/users just not working properly with scripts

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

By the end of this follow-up guide, I hope to be able to fix any issues with the initial setup, along with showing any useful information or tips that I discover along the way

## Fix 1: Enabling remote access to all systems

During the attacks, we'll require a way to gain remote access to the compromised systems, either for pivoting purposes or to further enumerate the system for valuable information. To do so, we can either use WinRM or RDP. For this demonstration, we'll be enabling both services on all Windows systems.&#x20;

To enable WinRM, we can create a new GPO to enable and configure certain settings for the WinRM services and clients

<figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

On each of the Windows systems, the WinRM QuickConfig utility must also be executed, in order to setup the service and enable firewall rules to permit access. After running the command, you can just click through and accept all changes requested, then complete the config

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption><p>Unfortunately I already completed config before writing this post</p></figcaption></figure>

To test WinRM connectivity, we can use these two commands&#x20;

```powershell
Test-WSMan 172.16.100.10 -Authentication Negotiate
```

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

```bash
ssh ravin@webserver.northland.org -i id_ed25519 -L 13389:172.16.100.125:5985
evil-winrm -i 127.0.0.1 -u Administrator -p '5ecUR3_P@ssw0rd!' -P 13389
```

<figure><img src="../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

To configure RDP, you just have to add a user to the "Remote Desktop Users" group

<figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

## Fix 2: Getting System Error 5 when running commands

Unfortunately, I was not able to reproduce this issue. Some Googling suggests that it could possibly be due to a lack of permissions. If you encounter this issue, you can try to add the user to the Administrators built-in security group, and try again

## Fix 3: Connectivity Issues

During the tests of the network, I encountered several issues with the firewall, such as not being able to connect via RDP from the external network.&#x20;

<figure><img src="../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

To fix this issue, we simply remove the firewall from the network, and add a second network adapter on the Linux webserver. This makes the webserver into a network pivot point, and allow us a smoother path to pivot towards the internal network and the DC.

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

