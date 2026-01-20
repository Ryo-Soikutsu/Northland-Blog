---
description: 'Solved by: Ryo Soikutsu (Hint from P3RP3LX)'
---

# pretty\_devilish\_file

## Challenge Description

{% code overflow="wrap" %}
```
Here is a little change of pace for us, but still within our area of expertise. Every know and then we have to break apart some busted document file to scoop out the goodies. Now it is your turn.
```
{% endcode %}

Category: Steganography

## Walkthrough

Extracting the zip file, we are provided with a PDF.

<figure><img src="../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

Analyzing the PDF, it appears to be completely normal. There is nothing out of the ordinary with it.

<figure><img src="../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

Switching to more specialized PDF tools though, we see something interesting

<figure><img src="../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

The PDF is encrypted, but we can still view the contents. What is going on? Using qpdf, we can write out the PDF in a JSON format, and notice the following contents

<figure><img src="../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

There is a chunk of encrypted contents in the PDF that does not appear in the preview. We can attempt to decrypt the encrypted chunk using qpdf too, using this command

<figure><img src="../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

Up until now, the tool has constantly warned us about the PDF file being damaged. To repair it, we can use PDFTK to repair our PDF.&#x20;

<figure><img src="../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

Using PDFTK, we can repair our PDF so that it won't return any more warnings. We can validate that the repair worked by attempting to output the PDF as a JSON file again. Notice that no warning messages were returned, compared to the original PDF.

Now, we can uncompress the data streams using qpdf again. At this point, it took a while to figure out the final step. After a bit of trial and error, it turns out that we can use PDFimages to extract any images contained in the PDF, and save it locally.

<figure><img src="../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

We can run strings on the image file, and reconstruct the flag manually

<figure><img src="../../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

## Conclusion

This took a long time to figure out, and caused a lot of frustration. I was fortunate to have a friend who had managed to solve this challenge, and gave me a hint to get the flag at the last step.
