# CTF trivia server (w/ socat)

A trivia server is very useful when you want to do question-based challenges, but don't want to have to resort to challenge spam. A trivia server will typically be accessed via commandline using the Netcat command, such as shown below

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption><p>Example of a barebones trivia server</p></figcaption></figure>

To simplify the process, we can use a Python script combined with the socat utility (read more [here](https://www.redhat.com/en/blog/getting-started-socat)) to allow participants to connect to the server

Here is a very minimalistic server, which was used for the Secure The Network DFIR challenge series

```python
# server.py
import os
import json

with open("questions.json") as f:
    questions: list = json.load(f)["questions"]

print("Logged into the Northland Forensics server.")
print("Answer all questions to complete the assignment.")

for i, question in enumerate(questions):
    print(f"\nQuestion {i+1}: {question['question']}")

    answer = input("> ").lower().strip()
    if answer != question["answer"]:
        print("\n\x1b[31mIncorrect answer. Exiting...\x1b[0m")
        exit(1)

print("All questions answered correctly. Assignment marked as completed.")
print(f"\x1b[32mAdmin message: {os.getenv("FLAG")}\x1b[0m")

```

It was accompanied by a JSON file containing the question bank, as well as the Dockerfile to spin up the server

```json
// questions.json
{
    "questions": [
        {
            "question": "What is the IP address of the attacker?",
            "answer": "201.172.32.43"
        },
        {
            "question": "What is the public IP address of the target?",
            "answer": "201.172.32.234"
        },
        {
            "question": "What tool did the attacker likely use to port scan the target? Answer in all lower-case",
            "answer": "nmap"
        },
        {
            "question": "What is the vulnerability used by the attacker on the target? Answer in all lower-case",
            "answer": "path traversal"
        },
        {
            "question": "What is the first directory enumeration tool that the attacker uses? Answer in all lower-case",
            "answer": "dirbuster"
        },
        {
            "question": "The attacker attempted to access the web server through SSH. What is the timestamp for the first successful access? Answer in UTC (Format is YYYY-MM-DD HH:MM:SS)",
            "answer": "2024-11-15 11:47:17"
        },
        {
            "question": "The attacker attempted to erase any traces of their attack by deleting system logs. Which files were modified by the attacker Answer in filename 1_filename 2",
            "answer": "auth.log_error.log"
        },
        {
            "question": "Upon further analysis of our firewall logs, it seems like one of the sysadmins had left in a rule which allowed for the attacker to access the remote server through SSH. Identify this overly-permissive rule. Answer is the rule's tracker ID",
            "answer": "1730707767"
        }
    ]
}
```

<pre class="language-dockerfile"><code class="lang-dockerfile"><strong># Dockerfile
</strong><strong>FROM python:3.12-alpine
</strong>WORKDIR /usr/src/physical

RUN apk add --no-cache socat

COPY . .

RUN addgroup -S physgrp &#x26;&#x26; adduser -S physuser -G physgrp
RUN chown -R physuser:physgrp /usr/src/physical
USER physuser

ENV FLAG=FLAG_REDACTED_FOR_PRIVACY

EXPOSE 1337
CMD ["socat", "-dd", "TCP-LISTEN:1337,fork,reuseaddr", "EXEC:python3 server.py"]

</code></pre>

The server can be built and ran using the following commands

```bash
docker build -t server .
docker run -dp 1337:1337 --name server server
```

Do note that on networks with very restrictive firewalls (e.g. school or corporate networks), access to the socat server may be blocked, as connections are not made via the common HTTP/HTTPS and DNS ports. If you anticipate that many participants will be connected to such networks, you may wish to consider using a webpage instead, or revert to using flag-based challenges instead

### Edit 2025-07-15

I have modified the original trivia server script to be much more lively and interesting. You can get the source code for the new server [here](https://github.com/Ryo-Soikutsu/Secure-The-Network/tree/main/Tools/Misc/TriviaServer)

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>
