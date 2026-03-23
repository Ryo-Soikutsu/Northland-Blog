---
description: 'Category: Web (Hard)'
---

# El Eff Aye

## Description

{% code overflow="wrap" %}
```
I found this old webapp running on one of the company's decommissioned servers. I wonder if it contains anything interesting on it...

Mirror 1: http://chall1.lagncra.sh:18631/

Mirror 2: http://chall2.lagncra.sh:18631/
```
{% endcode %}

## Walkthrough

Navigating to the website, we are greeted with a (fairly obviously vibecoded) website advertising an international competition.

<figure><img src="../../.gitbook/assets/image (352).png" alt=""><figcaption></figcaption></figure>

There isn't much on this site. There is a dropdown to change the language to Chinese or Japanese, and a "Join Now" button that requires a verification code to pass.&#x20;

<figure><img src="../../.gitbook/assets/image (353).png" alt=""><figcaption></figcaption></figure>

Going back to the home page and selecting a language, we can immediately see something interesting.

<figure><img src="../../.gitbook/assets/image (354).png" alt=""><figcaption></figcaption></figure>

A new parameter has been added to the URL, which seems to point to `lang/cn.php`. This parameter contains the path that the webserver should load for a specific language, such as `lang/jp.php` for Japanese or `lang/en.php` for English. If this parameter is not safely sanitized, it could lead to a potential local file inclusion attack, allowing a malicious actor to read files they're not supposed to.

We can try to read the source code for `verify.php` which we found earlier

<figure><img src="../../.gitbook/assets/image (355).png" alt=""><figcaption></figcaption></figure>

Thats not what we want! We want to see the source code itself, not the final rendered version. We can use a trick called PHP wrappers, which allow us to use a pre-set list of instructions to do various things, like reading files or running code. We'll use the `php://` wrapper, which will allow us to read a specified file and perform data transforming, like encoding it in Base64. We can use the following payload in order to read the contents of `verify.php`

```
php://filter/convert.base64-encode/resource=/var/www/html/verify.php
```

* If you're wondering where I got the `/var/www/html`, it is one of the common webroot directories that webservers tend to use.&#x20;

<figure><img src="../../.gitbook/assets/image (356).png" alt=""><figcaption></figcaption></figure>

We have the Base64-encoded version of the page, which we can now decode to get the source code. The final source code for `verify.php` has been provided below

```php
<?php
session_start();
$result = null;
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $code = $_POST['code'] ?? '';
    if (preg_match('/^[A-Z]{4}(_\d{4}){3}$/', $code)) {
      $flag = getenv('FLAG');
      $result = ['success', "Valid verification code. Here is the flag: $flag"];
    } else {
      $result = ['error', 'Invalid verification code. Please submit a valid code.'];
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Verification</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        body {
            margin: 0;
            font-family: system-ui, -apple-system, BlinkMacSystemFont, sans-serif;
            background: #0f172a;
            color: #e5e7eb;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        .card {
            background: #020617;
            padding: 32px;
            border-radius: 10px;
            width: 100%;
            max-width: 400px;
        }

        h2 {
            margin-top: 0;
            text-align: center;
        }

        input {
            width: 100%;
            padding: 10px;
            margin-top: 12px;
            background: #020617;
            color: #e5e7eb;
            border: 1px solid #334155;
            border-radius: 6px;
        }

        button {
            width: 100%;
            padding: 10px;
            margin-top: 16px;
            background: #2563eb;
            color: white;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-weight: 600;
        }

        button:hover {
            background: #1d4ed8;
        }

        .message {
            margin-top: 16px;
            padding: 10px;
            border-radius: 6px;
            text-align: center;
        }

        .success {
            background: #064e3b;
            color: #a7f3d0;
        }

        .error {
            background: #7f1d1d;
            color: #fecaca;
        }
    </style>
</head>
<body>
<div class="card">
    <h2>Verification</h2>
    <p>Please enter your verification code.</p>

    <form method="post">
        <input
            type="text"
            name="code"
            placeholder="Enter your verification code here."
            required
        >
        <button type="submit">Verify</button>
    </form>
    <?php if ($result): ?>
        <div class="message <?= $result[0] ?>">
            <?= htmlspecialchars($result[1]) ?>
        </div>
    <?php endif; ?>
</div>
</body>
</html>
```

The verification system uses regex to match for a code that matches the following requirements

* Begins with a 4-character uppercase alphabetical code
* Contains 3 blocks of 4 numerical numbers, separated with an underscore
* E.g. ABCD\_1111\_1111\_1111 will pass the filter

We can enter the example code, and it will successfully verify us

<figure><img src="../../.gitbook/assets/image (357).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (362).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

Nothing much to say, I got inspiration for this challenge after doing HackTheBox's Web Attacks module.&#x20;
