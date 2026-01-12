---
description: AlligatorPay
---

# Level 4

## Description

{% code overflow="wrap" %}
```
In the dark corners of the internet, whispers of an elite group of hackers aiding our enemies have surfaced. The word on the street is that a good number of members from the elite group happens to be part of an exclusive member tier within AlligatorPay (agpay), a popular payment service.


Your task is to find a way to join this exclusive member tier within AlligatorPay and give us intel on future cyberattacks. AlligatorPay recently launched an online balance checker for their payment cards. We heard it's still in beta, so maybe you might find something useful.
```
{% endcode %}

_Author's Note: This was an extremely easy challenge to solve using ChatGPT. However, for the sake of the writeup, I will try to explain the steps and exploit myself, so that readers who are more curious about the solve process won't just be stuck with "Ask ChatGPT" as the solution_

Category: Cryptography, Reverse Engineering, Web Exploitation

## Walkthrough

This challenge was a really easy challenge compared to the previous levels. I woke up expecting to spend another day on it, but ended up taking only 20 minutes with the help of ChatGPT.

Visiting the URL given, we are presented with the AlligatorPay webpage

<figure><img src="../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

Being a web challenge, I searched the usual robots.txt, sitemap.xml and /admin endpoint, but got nothing from it. From there, I used Inspect Element to take a look at the client-side source code for the webpage, and managed to get a couple of hints to solve this challenge.

<figure><img src="../../.gitbook/assets/image (109).png" alt=""><figcaption><p>Looks like we have a test card to play with, and a target balance to hit</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (110).png" alt=""><figcaption><p>Part of the source code</p></figcaption></figure>

I copied the JS script out to VS Code, and cleaned it up to only include the important bit (which thankfully made up more than half of the code). If we took a look at the code, it seemed like the website takes in a digital card file, checks for a certain start and end string, and then decodes the file to get the account data

```javascript
 // Checks the file for a starting "AGPAY" string
 // and ending "ENDAGP" string
 const signature = getString(dataView, 0, 5);
  if (signature !== "AGPAY") {
    alert("Invalid Card");
    return;
  }
  const version = getString(dataView, 5, 2);
  const encryptionKey = new Uint8Array(arrayBuffer.slice(7, 39));
  const reserved = new Uint8Array(arrayBuffer.slice(39, 49));

  const footerSignature = getString(dataView, arrayBuffer.byteLength - 22, 6);
  if (footerSignature !== "ENDAGP") {
    alert("Invalid Card");
    return;
  }
```

```javascript
// Decrypts the data and extracts important info
  const encryptionKey = new Uint8Array(arrayBuffer.slice(7, 39));
  const checksum = new Uint8Array(
    arrayBuffer.slice(arrayBuffer.byteLength - 16, arrayBuffer.byteLength)
  );

  const iv = new Uint8Array(arrayBuffer.slice(49, 65));
  const encryptedData = new Uint8Array(
    arrayBuffer.slice(65, arrayBuffer.byteLength - 22)
  );

  const calculatedChecksum = hexToBytes(
    SparkMD5.ArrayBuffer.hash(new Uint8Array([...iv, ...encryptedData]))
  );

  if (!arrayEquals(calculatedChecksum, checksum)) {
    alert("Invalid Card");
    return;
  }

  const decryptedData = await decryptData(encryptedData, encryptionKey, iv);

  const cardNumber = getString(decryptedData, 0, 16);
  const cardExpiryDate = decryptedData.getUint32(20, false);
  const balance = decryptedData.getBigUint64(24, false);

```

We can determine that the key indexes of the file is as follows:

* Encryption key: indexes 7 to 39 (length: 32)
* Encrypted data: indexes 65 to 113 (length: 48)

From the decrypted data, the key indexes are as follows:

* Card number: indexes 0 to 16 (length: 16)
* Expiry date: indexes 20 to 24 (length: 4)
* Balance: indexes 24 to 32 (length: 8)

Knowing the indexes that we'll have to extract later on, we can focus on the decryption feature itself

```javascript
async function decryptData(encryptedData, key, iv) {
  const cryptoKey = await crypto.subtle.importKey(
    "raw",
    key,
    { name: "AES-CBC" },
    false,
    ["decrypt"]
  );
  const decryptedBuffer = await crypto.subtle.decrypt(
    { name: "AES-CBC", iv: iv },
    cryptoKey,
    encryptedData
  );
  return new DataView(decryptedBuffer);
}
```

This was a very simple script, using the AES encryption algorithm with CBC mode to import the key from the file, then decrypting the data. It then returns an array containing the decrypted data. With this information, we can now write a solve script to decrypt the file and get the plaintext data.

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
import hashlib

def decrypt_data(encrypted_data, key, iv):
    """Decrypt the encrypted data using AES-CBC."""
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
    decryptor = cipher.decryptor()
    decrypted_data = decryptor.update(encrypted_data) + decryptor.finalize()
    return decrypted_data

def validate_checksum(decrypted_data, checksum):
    """Validate checksum using MD5."""
    calculated_checksum = hashlib.md5(decrypted_data).digest()
    return calculated_checksum == checksum

def read_binary_file(file_path):
    """Read the binary file and extract components."""
    with open(file_path, 'rb') as f:
        file_data = f.read()

    # Extracting components based on known positions
    signature = file_data[:5].decode('utf-8')
    version = file_data[5:7].decode('utf-8')
    encryption_key = file_data[7:39]
    reserved = file_data[39:49]
    iv = file_data[49:65]
    encrypted_data = file_data[65:-22]
    footer_signature = file_data[-22:-16].decode('utf-8')
    checksum = file_data[-16:]

    return signature, version, encryption_key, iv, encrypted_data, footer_signature, checksum

def format_card_number(card_number):
    """Format the card number into readable groups."""
    return ' '.join([card_number[i:i+4] for i in range(0, len(card_number), 4)])

def format_date(expiry_timestamp):
    """Format the card expiry date from timestamp."""
    from datetime import datetime
    expiry_date = datetime.fromtimestamp(expiry_timestamp)
    return expiry_date.strftime("%m/%y")

def main(file_path):
    # Read binary file and extract components
    signature, version, encryption_key, iv, encrypted_data, footer_signature, checksum = read_binary_file(file_path)

    if signature != "AGPAY" or footer_signature != "ENDAGP":
        print("Invalid Card")
        return

    # Decrypt the data
    decrypted_data = decrypt_data(encrypted_data, encryption_key, iv)

    # Validate checksum

    # Parse decrypted data
    card_number = decrypted_data[:16].decode('utf-8')
    expiry_date = int.from_bytes(decrypted_data[20:24], 'big')  # Assuming expiry date is at byte 20-23
    balance = int.from_bytes(decrypted_data[24:32], 'big')  # Assuming balance is at byte 24-31

    # Output results
    print("Card Number:", format_card_number(card_number))
    print("Expiry Date:", format_date(expiry_date))
    print("Balance: $", balance)

if __name__ == "__main__":
    file_path = "payload.agpay"  # Replace with your binary file path
    main(file_path)
```

The following script reads the binary data of the AGPay card file, and extracts the encryption key and data before performing decryption on it to retrieve the original data. Running the previous script, we can see that we get the following data:

<figure><img src="../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

The data that is extracted matches the data that we saw earlier from the website. In order to solve this challenge, we need to have a balance of $313371337 in our card. To do that, we can write a script to perform the reverse and generate a payload card with the respective balance amount

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
import hashlib
import os
import struct
from datetime import datetime

def encrypt_data(data, key, iv):
    """Encrypt the data using AES-CBC."""
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
    encryptor = cipher.encryptor()
    encrypted_data = encryptor.update(data) + encryptor.finalize()
    return encrypted_data

def generate_checksum(data):
    """Generate an MD5 checksum for the data."""
    return hashlib.md5(data).digest()

def generate_card_file(card_number, expiry_date, balance, encryption_key, iv, output_file):
    """Generate a card file with encrypted data and other components."""
    # Convert the card number, expiry date, and balance to binary
    card_number_padded = card_number.ljust(16, '\0').encode('utf-8')
    expiry_timestamp = int(expiry_date.timestamp())
    expiry_bytes = struct.pack('>I', expiry_timestamp)
    balance_bytes = struct.pack('>Q', balance)

    # Prepare the data to be encrypted (card number, expiry date, balance)
    plaintext = card_number_padded + b'\0' * 4 + expiry_bytes + balance_bytes

    # Encrypt the data
    encrypted_data = encrypt_data(plaintext, encryption_key, iv)

    # Calculate checksum
    checksum = generate_checksum(iv + encrypted_data)

    # File structure (example)
    signature = "AGPAY".encode('utf-8')  # 5 bytes
    version = "01".encode('utf-8')  # 2 bytes
    reserved = b'\0' * 10  # 10 bytes reserved
    footer_signature = "ENDAGP".encode('utf-8')  # 6 bytes

    # Create binary content
    card_file_content = (
        signature +
        version +
        encryption_key +
        reserved +
        iv +
        encrypted_data +
        footer_signature +
        checksum
    )

    # Write the binary content to a file
    with open(output_file, 'wb') as f:
        f.write(card_file_content)

    print(f"Card file generated: {output_file}")

def get_user_input():
    """Prompt the user for card information."""
    card_number = input("Enter card number (16 digits): ").strip()
    if len(card_number) != 16:
        raise ValueError("Card number must be 16 digits long")

    expiry_str = input("Enter expiry date (MM/YY): ").strip()
    expiry_date = datetime.strptime(expiry_str, "%m/%y")

    balance = int(input("Enter balance (as integer): ").strip())

    return card_number, expiry_date, balance

if __name__ == "__main__":
    card_number, expiry_date, balance = get_user_input()
    encryption_key = os.urandom(32)
    iv = os.urandom(16)
    output_file = "payload.agpay"
    generate_card_file(card_number, expiry_date, balance, encryption_key, iv, output_file)

```

A relatively simple script, it takes in a set of user inputs for the card number, expiry date and account balance, then outputs a new card with those values. It uses the Python `struct` library to work with raw binary data, since for some reason Python cannot write binary data directly to files by itself. With this generated file, we upload the payload card to the website and try to get the flag.

<figure><img src="../../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

Unfortunately, our card returns an error message, too vague to get much information out of (thanks JavaScript). After a few minutes of trial and error, I figured out that the file was missing some padding that could have caused the webapp to error out. In particular, I was supposed to implement PCKS7 padding to the binary data, in order for the indexes of the data to fit properly (I'm probably tripping, but thats the best reasoning my half-asleep brain could come up with). Asking ChatGPT to modify the original script to include the new padding, I was returned the following:

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import padding
import hashlib
import os
import struct
from datetime import datetime

def pad_data(data):
    """Pad the data using PKCS7 to ensure block size alignment (16 bytes for AES)."""
    padder = padding.PKCS7(algorithms.AES.block_size).padder()  # AES block size is 128 bits (16 bytes)
    padded_data = padder.update(data) + padder.finalize()
    return padded_data

def encrypt_data(data, key, iv):
    """Encrypt the data using AES-CBC with PKCS7 padding."""
    # Apply PKCS7 padding
    padded_data = pad_data(data)

    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
    encryptor = cipher.encryptor()
    encrypted_data = encryptor.update(padded_data) + encryptor.finalize()
    return encrypted_data

def generate_checksum(data):
    """Generate an MD5 checksum for the data."""
    return hashlib.md5(data).digest()

def generate_card_file(card_number, expiry_date, balance, encryption_key, iv, output_file):
    """Generate a card file with encrypted data and other components."""
    # Convert the card number, expiry date, and balance to binary
    card_number_padded = card_number.ljust(16, '\0').encode('utf-8')
    expiry_timestamp = int(expiry_date.timestamp())
    expiry_bytes = struct.pack('>I', expiry_timestamp)  # Big-endian 4-byte integer
    balance_bytes = struct.pack('>Q', balance)  # Big-endian 8-byte integer

    # Prepare the data to be encrypted (card number, expiry date, balance)
    plaintext = card_number_padded + b'\0' * 4 + expiry_bytes + balance_bytes

    # Encrypt the data
    encrypted_data = encrypt_data(plaintext, encryption_key, iv)

    # Calculate checksum
    checksum = generate_checksum(iv + encrypted_data)

    # File structure (example)
    signature = "AGPAY".encode('utf-8')  # 5 bytes
    version = "01".encode('utf-8')  # 2 bytes
    reserved = b'\0' * 10  # 10 bytes reserved
    footer_signature = "ENDAGP".encode('utf-8')  # 6 bytes

    # Create binary content
    card_file_content = (
        signature +
        version +
        encryption_key +
        reserved +
        iv +
        encrypted_data +
        footer_signature +
        checksum
    )

    # Write the binary content to a file
    with open(output_file, 'wb') as f:
        f.write(card_file_content)

    print(f"Card file generated: {output_file}")

def get_user_input():
    """Prompt the user for card information."""
    card_number = "1234567890123456"  # input("Enter card number (16 digits): ").strip()
    if len(card_number) != 16:
        raise ValueError("Card number must be 16 digits long")

    expiry_str = "05/24"
    expiry_date = datetime.strptime(expiry_str, "%m/%y")

    balance = 313371337

    return card_number, expiry_date, balance

if __name__ == "__main__":
    card_number, expiry_date, balance = get_user_input()

    # Generate random encryption key and IV (16 bytes for AES-128)
    encryption_key = os.urandom(32)  # AES-256 requires 32-byte key
    iv = os.urandom(16)  # 16-byte IV for AES-CBC

    # Output file path (you can customize this)
    output_file = "payload.agpay"

    # Generate the card file
    generate_card_file(card_number, expiry_date, balance, encryption_key, iv, output_file)

```

The new script just adds a pad\_data function that takes in the plaintext data, adds some padding bits to it, then performs the encryption process.

This time, when we upload our payload card, we get our flag

<figure><img src="../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>
