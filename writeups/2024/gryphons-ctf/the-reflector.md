---
description: 'Solved by: Ryo Soikutsu'
---

# The Reflector

## Challenge Description

{% code overflow="wrap" %}
```
This website features a "debug" page intended for internal use only. It reflects back any GET parameters you send. However, access is strictly restricted. Can you bypass the restrictions?
```
{% endcode %}

Category: Web (Hard)

## Walkthroug

Looking at the webpage, it seems like there's only one endpoint, /debug. However, when we navigate to it, we get an Access Denied response

<figure><img src="../../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>

Looking at the source code, we can see that the endpoint performs authentication based on the request headers, and checks if the value for X-Forwarded-For is set to 127.0.0.1 (localhost)

```python
@app.route("/debug")
def debug():
    if request.headers.get("X-Forwarded-For") != "127.0.0.1":
        return "Access Denied", 403

    params = request.args.to_dict()
    if not params:
        return render_template("debug.html")

    forbidden_chars = ["<", ">", '"', "'", "[", "]"]

    for key, value in params.items():
        for char in forbidden_chars:
            if char in value:
                return "Forbidden character used!", 400

    reflected_output = ""
    for key, value in params.items():   
        reflected_output += f"{key} = {value}<br>"

    try:
        rendered_output = render_template_string(reflected_output)
        return rendered_output
    except Exception:
        return "Template Error", 500
```

By setting the header in our curl request, we can now see that we are authenticated

<figure><img src="../../.gitbook/assets/image (162).png" alt=""><figcaption></figcaption></figure>

The code has an XSS and SSTI vulnerability, as it directly renders user input without any validation. While the value argument is being sanitized against a blacklist, the key argument isn't, so injection can be performed on that

Using a standard test payload for SSTI injection ( \{{7\*7\}} ), we can see that it is indeed vulnerable to SSTI injection

<figure><img src="../../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

Now we just copy a payload from another writeup :P&#x20;

Payload:&#x20;

```python
{{request.application.__globals__.__builtins__.open('flag.txt').read()}}
```

<figure><img src="../../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>
