# Infrastructure Setup

This section of the guide will go over the steps for setting up the infrastructure used to develop the challenge series. We'll be using VMware Workstation Pro for the rest of this guide, as well as the following ISOs:

* Kali Linux
* Ubuntu Server
* Windows 10
* Windows Server 2022
* Netgate PFSense

It is recommended to create a base VM that you can create clones from, instead of having to create a new virtual machine each time.&#x20;

## Network Overview

We'll be setting up the network as such

<figure><img src="../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

The PCs labelled as "Communications" will be used to simulate conversations and chats between two parties. The network topology may change between challenges, but this will serve as a good starting point.

## Setting Up the Network

For this section, we'll only be doing basic configuration, before we take a snapshot of the VMs. This will allow us to be able to revert back changes in the event that they fail, or when we want to make new challenges. We will be configuring the DC, employee PC, web server, as well as an (inaccessible) Wazuh instance to forward logs to

The details for the systems being configured are as such

| Workstation Name              | Network | Details                                                                          |
| ----------------------------- | ------- | -------------------------------------------------------------------------------- |
| DC01.northland.org            | LAN     | <p>IP address: 172.16.100.125/24<br>Domain: northland.org</p>                    |
| DESKTOP-4V3RT12.northland.org | LAN     | IP address: DHCP                                                                 |
| DESKTOP-8G1R36M.northland.org | LAN     | IP address: DHCP                                                                 |
| webserver.northland.org       | DMZ     | IP address: 10.129.25.224                                                        |
| FW1                           | -       | <p>WAN: 120.225.134.22/24<br>LAN: 172.16.100.254/24<br>DMZ: 10.129.25.254/24</p> |

| Username                       | Password          | Workstation Name                          |
| ------------------------------ | ----------------- | ----------------------------------------- |
| Administrator                  | 5ecUR3\_P@ssw0rd! | AD / AD DSRM                              |
| Solitude (Local Administrator) | 5ecUR3\_P@ssw0rd! | <p>DESKTOP-8G1R36M<br>DESKTOP-4V3RT12</p> |
| Root                           | 5ecUR3\_P@ssw0rd! | webserver.northland.org                   |
| kali                           | kali              | Kali attacker                             |

### Setting Up DC01.northland.org

<figure><img src="../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

Simply click "Next" for the rest of the prompts, and click "Install" when prompted to. We'll also be creating some users for our attacker to enumerate and escalate privileges to. We'll be creating three organizational units (OU), IT, HR and executive. We'll also be creating some users and assigning them to their appropriate groups

| Full Name         | User Logon Name | Password     | OU        |
| ----------------- | --------------- | ------------ | --------- |
| Ryo Soikutsu      | ryo.soikutsu    | P@ssw0rd123  | Executive |
| Kevin Snow        | kevin.snow      | letitgo13!   | Executive |
| Matias Torres     | matias.torres   | Alicorn1!    | HR        |
| Mihaly A. Shilage | mihaly.shilage  | 4rchange#    | HR        |
| Schroeder         | schroeder       | 1l0vEDr0n3S! | IT        |

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

(Yeah, I might've played a bit too much Ace Combat during the course of making this guide...)

Finally, we'll configure some Group Policy Objects (GPO) to enable auditing and install Sysmon.&#x20;

<figure><img src="../../.gitbook/assets/image (56).png" alt=""><figcaption><p>Configure the policies circled in red</p></figcaption></figure>

To install Sysmon, follow the steps outlined in this article ([link](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)).

That concludes configuration for the DC. To configure the client machines (DESKTOP-\*), just configure them to join the northland.org domain, and install Sysmon on them too.

### Setting up FW1

This was a tricky one to set up at first. The ISO for the firewall can be installed from Netgate's website [here](https://www.pfsense.org/download). When creating the VM, make sure to add and configure the network adapters one at a time, otherwise there will be a mix-up of the adapters and the firewall will not work. Adapters will be set up in the following order

<figure><img src="../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

Once the network adapters are set up, you can connect to the web interface by accessing the LAN interface. Follow the setup wizard to complete initial setup.

<figure><img src="../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

### Setting up Webserver.northland.org

For Linux VMs, I typically set up my network adapters as such

<figure><img src="../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

This allows me to install and update packages as needed in the event that some packages are missing during initial setup

Setup for the webserver can be easily automated using a Bash script like this. Run the script as root

```bash
hostnamectl set-hostname webserver.northland.org
nmcli dev up ens224 # Change this interface to the interface on your VM that is disabled
nmcli conn mod ens224 ipv4.method manual ipv4.addr 10.129.25.224/24 ipv4.gate 10.129.25.254
nmcli dev reapply ens224

dnf check-update -y
reboot
```

## Conclusion

That concludes the basic configuration of the entire network. In the next section, we'll continue with specialized configurations for Secure The Network 2 machines.
