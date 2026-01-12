---
description: 'Creator: scuffed'
---

# lovebomb!

## Description

```
you have no taste 👅
```

Category: Reverse Engineering (Easy)

## Walkthrough

We are provided with an interesting Python script, as shown below

```python
i_love_u = """
🧡💜💜
💙🤎
🤍💜
💙🤎
🖤💛
🖤💜
🖤🖤
🧡💛
🧡💛
🧡💙💙

<REDACTED FOR BREVITY>
"""

def i_love_u_i_love_u(loove):
    ily = []
    love = loove.split('\n')[1:-1]
    for hearts in love:
        looove = "".join(str('🤎🧡💛💚💙💜🖤🤍'.index(heart)) for heart in hearts)
        yayyy = int(looove, (1<3)+(1<3)+(1<3)+(1<3)+(1<3)+(1<3)+(1<3)+(1<3))
        ily.append(chr(yayyy))
    my_love = "".join(ily)
    exec(my_love, globals())

i_love_u_i_love_u(i_love_u)

```

A massive portion of the script is filled with various heart icons, and at the bottom is a Python program that will parse the hearts and convert them into ASCII characters. Afterwards, the program will execute the converted code. We can easily get the code by replacing the exec command with a print statement instead, which will write the contents of <mark style="color:red;">my\_love</mark> out to the console.

```python
# Second stage
m = 256

def K(k):
    S = list(range(m))
    j = 0
    for i in range(m):
        j = (j + S[i] + k[i % len(k)]) % m
        S[i], S[j] = S[j], S[i]
    return S

def P(S):
    i = 0
    j = 0
    while True:
        i = (i + 1) % m
        j = (j + S[i]) % m
        S[i], S[j] = S[j], S[i]
        K = S[(S[i] + S[j]) % m]
        yield K

def decrypt(k):
    c = b'-\x9b\xf3S\xab\xdd0\x81\xcf\xf3\xda\x87Mn\xc4\xeel\x80\xa1+#vu\xa2\xbd\x93\xf9\x9b\x9a\xff@C\x8f\x1a\x19=\xae\x0f/'
    k = [ord(c) for c in k]
    ks = P(K(k))
    r = ""
    for C in c:
        r += (chr(C ^ next(ks)))
    return r

k = input("What's the password? ")
if k == '7h1515mys3cr37':
    d = decrypt(k)
    print('Flag:', d)
else:
    print("Nice try!")
```

The second stage will check if the user has provided the correct password. If the user does provide the correct password, the program will decrypt a hexadecimal string using an RC4-like stream cipher algorithm. Fortunately, we don't need to reverse the script completely, as we can just pass in the password to have the program automatically decrypt it for us.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption><p>Think someone watched the Chainsaw Man movie</p></figcaption></figure>
