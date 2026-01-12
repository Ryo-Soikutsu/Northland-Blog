---
description: 'Solved by: Ryo Soikutsu'
---

# Not Hexadecimal

## Challenge Description

```
Seriously, this is not hexadecimal, I promise.
```

Category: OSINT (Easy)

## Walkthrough



This is a relatively easy OSINT challenge (despite the low solve count at the time of writing). We are given a file (called "<mark style="color:red;">secret.txt</mark>") which contains 3 lines of alphanumeric characters. At first glance, they appear to be simply hexadecimal characters

<figure><img src="../../.gitbook/assets/image (158).png" alt=""><figcaption><p>Writing the contents of secret.txt to the terminal</p></figcaption></figure>

These strings correspond to cell IDs on Uber's H3 geographical navigation system. Public access to the map can be found [here](https://h3geo.org). Plugging the 3 values into the website gives us these three cells, and this central point

<figure><img src="../../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>



## Conclusion

* In the event that H3 IDs need to be converted to coordinates, the Python H3 library can be used instead of having to search manually in Google Maps

```python
import h3
h3.cell_to_latlng("cell_id")
```

