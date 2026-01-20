---
description: 'Creator: Ravin'
---

# Silly Goose On The Loose

## Description

{% code overflow="wrap" %}
```
A silly goose on the loose, Waddling 'round with no excuse. Flapping wings, so wild and free, Spreading chaos for all to see
```
{% endcode %}

Category: Reverse Engineering (Easy)

## Walkthrough

We are provided with a single-page PDF file, containing a goose wearing a funny hat and shoes

<figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

Checking various PDF metadata, we don't see anything unusual with the file, except for the presence of JavaScript.&#x20;

<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption><p>Command: qpdf --json sillygooseontheloose.pdf</p></figcaption></figure>

This is the JavaScript after cleaning it up&#x20;

```javascript
(function () {
    const _0xcafe = {
        'a': 0x48, 'b': 0x4e, 'c': 0x46, 'd': 0x32, 'e': 0x35,
        'f': 0x7b, 'g': 0x69, 'h': 0x6d, 'i': 0x5f, 'j': 0x6a,
        'k': 0x75, 'l': 0x35, 'm': 0x74, 'n': 0x5f, 'o': 0x61,
        'p': 0x5f, 'q': 0x73, 'r': 0x31, 's': 0x6c, 't': 0x6c,
        'u': 0x79, 'v': 0x5f, 'w': 0x67, 'x': 0x30, 'y': 0x30,
        'z': 0x73, 'A': 0x65, 'B': 0x2e, 'C': 0x2e, 'D': 0x2e,
        'E': 0x2e, 'F': 0x2e, 'G': 0x30, 'H': 0x6e, 'I': 0x5f,
        'J': 0x74, 'K': 0x68, 'L': 0x33, 'M': 0x5f, 'N': 0x6c,
        'O': 0x30, 'P': 0x73, 'Q': 0x65, 'R': 0x7d
    };

    const _0xdead = function (_0x1337) {
        let _0xbabe = '';
        for (let _0xc0de = 0; _0xc0de < _0x1337.length; _0xc0de++) {
            _0xbabe += String.fromCharCode(_0xcafe[_0x1337[_0xc0de]]);
        }
        return _0xbabe;
    };

    const _0xfeed = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQR'.split('');

    const _0xbeef = function (_0x8bad) {
        const _0x9cab = [0x67, 0x6f, 0x6f, 0x73, 0x65];
        let _0xface = 0;

        for (let _0xacdc = 0; _0xacdc < _0x9cab.length; _0xacdc++) {
            _0xface += _0x9cab[_0xacdc];
        }

        if (_0xface === 0x1f7) {
            const _0xfade = _0xdead(_0xfeed.join(''));

            const _0xb0b = function (_0xstr) {
                let _0xtmp = '';
                for (let _0xi = 0; _0xi < _0xstr.length; _0xi += 2) {
                    if (_0xi + 1 < _0xstr.length) {
                        _0xtmp += _0xstr[_0xi + 1] + _0xstr[_0xi];
                    } else {
                        _0xtmp += _0xstr[_0xi];
                    }
                }
                return _0xtmp;
            };

            const _0xdada = _0xb0b(_0xfade);

            const _0xf1a9 = function (_0xinput) {
                let _0xout = '';
                const _0xkey = [1, 0, 1, 0, 1, 0];
                for (let _0xidx = 0; _0xidx < _0xinput.length; _0xidx++) {
                    if (_0xkey[_0xidx % _0xkey.length] === 1) {
                        _0xout += _0xinput[_0xidx];
                    }
                }
                return _0xout;
            };

            const _0$c001 = function (_0xdata) {
                const _0xmap = {};
                for (let _0xa = 0; _0xa < _0xdata.length; _0xa++) {
                    _0xmap[_0xa] = _0xdata[_0xa];
                }
                let _0xresult = '';
                const _0xorder = Object.keys(_0xmap).sort((a, b) => {
                    const _0xmodA = parseInt(a) % 3;
                    const _0xmodB = parseInt(b) % 3;
                    return _0xmodA - _0xmodB;
                });
                _0xorder.forEach(_0xk => _0xresult += _0xmap[_0xk]);
                return _0xresult;
            };

            const _0xfinal = _0$c001(_0xdada);

            if (_0x8bad && typeof _0x8bad === 'function') {
                _0x8bad(_0xfinal);
            } else {
                console.log('🪿 Honk! You found me!');
                console.log(_0xfinal);
            }

            return _0xfinal;
        }

        return null;
    };

    _0xbeef();
})();

```

Much of the code is unnecessary, except for the top part of the script

```javascript
const _0xcafe = {
        'a': 0x48, 'b': 0x4e, 'c': 0x46, 'd': 0x32, 'e': 0x35,
        'f': 0x7b, 'g': 0x69, 'h': 0x6d, 'i': 0x5f, 'j': 0x6a,
        'k': 0x75, 'l': 0x35, 'm': 0x74, 'n': 0x5f, 'o': 0x61,
        'p': 0x5f, 'q': 0x73, 'r': 0x31, 's': 0x6c, 't': 0x6c,
        'u': 0x79, 'v': 0x5f, 'w': 0x67, 'x': 0x30, 'y': 0x30,
        'z': 0x73, 'A': 0x65, 'B': 0x2e, 'C': 0x2e, 'D': 0x2e,
        'E': 0x2e, 'F': 0x2e, 'G': 0x30, 'H': 0x6e, 'I': 0x5f,
        'J': 0x74, 'K': 0x68, 'L': 0x33, 'M': 0x5f, 'N': 0x6c,
        'O': 0x30, 'P': 0x73, 'Q': 0x65, 'R': 0x7d
    };

    const _0xdead = function (_0x1337) {
        let _0xbabe = '';
        for (let _0xc0de = 0; _0xc0de < _0x1337.length; _0xc0de++) {
            _0xbabe += String.fromCharCode(_0xcafe[_0x1337[_0xc0de]]);
        }
        return _0xbabe;
    };

    const _0xfeed = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQR'.split('');
    // ...
    // ...
    if (_0xface === 0x1f7) {
            const _0xfade = _0xdead(_0xfeed.join(''));
```

This code takes a string of characters and reassembles it into the flag by getting the mapped hexadecimal value of each character, then converting it to a character and appending it to the output string. For example, the character "a" will be mapped to 0x48, which will be mapped to the value "H"

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

We can directly copy-paste the chunk of code into DevTools' Console, and then print out the output of the function

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>
