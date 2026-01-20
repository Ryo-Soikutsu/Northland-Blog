---
description: 'Category: OSINT (Medium)'
---

# Traveling Intern 1

## Description

{% code overflow="wrap" %}
```
Welcome to the International Contract Agency, intern. Please refer to the report for today's assignment

Flag format is YBN{hotel_name_all_lowercase_no_spaces}. E.g. if the hotel name is Shangri-La Hotel, the flag will be YBN{shangri-la_hotel}
```
{% endcode %}

Assignment Brief:

{% code overflow="wrap" %}
```
First off, on behalf of the ICA Data Analysis Department, we welcome you as a new junior analyst. We hope that you’ll be able to adapt to our environment in no time.

Moving onto the assignment. We have received a new contract targeting the CISO of Kronstadt Industries, William Kronx. Unfortunately, we have not been able to track down his location so far, as he keeps no social media presence and never shares his location information with anyone outside of his trusted circle. 

Recently, we noticed that the CISO’s secretary has hired a new intern to take her place and has managed to convince William to allow the intern to accompany them during a company trip abroad. Fortunately, the intern has a bad sense of OPSEC, and frequently posts on their Instagram account. 

We know that the intern is a big fan of the CTF team Yes But No (YBN), and has even made it part of his social media usernames. Help us track the intern down and get intel on William’s whereabouts. Maybe you can find out what hotel they are staying at?
```
{% endcode %}

## Walkthrough

The assignment brief is pretty vague, giving little information to discover the intern's socials. From the assignment brief, we can glance at the following

* The intern has bad OPSEC and frequently posts, so maybe they'll accidentally leak location information through their posts
* The intern has been invited to follow William Knox on a company trip
* The intern has an Instagram account, narrowing down the search to one platform
* The intern is a fan of Yes But No (YBN), so they'll maybe include it somewhere on their profile
* (Not explicitly stated) The intern may also include the fact that they're an intern somewhere on their profile

For this challenge, we'll be using an OSINT method known as Google Dorking, which allows us to easily narrow down precise searches using a series of tags and keywords

<details>

<summary>Google Dorking Explained</summary>

**Google Dorking**, also known as **Google hacking**, is a technique that uses advanced search operators in the Google search engine to find specific, often sensitive, information that has been unintentionally exposed to the public internet. This information is already publicly available but not easily discoverable through normal search queries

There are several tags that are commonly used when performing Google Dorking

* **`site:`** Restricts search results to a specific website or domain (e.g., `site:example.com`).
* **`filetype:`** Filters results for a specific file type (e.g., `filetype:pdf` or `filetype:xlsx`).
* **`inurl:`** Finds pages that have the specified text within their URL (e.g., `inurl:admin`).
* **`intitle:`** Locates pages with the specified text in their HTML title tag (e.g., `intitle:"index of"`).
* **`intext:`** Searches for a specific phrase within the main body text of a page (e.g., `intext:"password"`).
* **`-` (minus operator)** Excludes results containing a specific word or phrase (e.g., `apple -Inc.`).
* **`"` (quotation marks)** Searches for an exact phrase (e.g., `"confidential records"`).&#x20;

</details>

Google Dorking can be hit or miss at times, so sometimes you may choose to search using incognito, or on other browsers. Google Dorking can also be unpredictable, so sometimes the same query may not display the same results, or even display results at all. An alternative is using Instagram's native search, although I have been told that it is not very reliable either.&#x20;

Assuming you did find the intern's page, you will see a couple of posts that the intern have made, supposedly during the company trip.

<figure><img src="../../.gitbook/assets/image (15).png" alt="" width="369"><figcaption><p>Had to switch to mobile because Instagram wouldn't stop prompting me to login >:(</p></figcaption></figure>

Looking at the posts, it seems like the intern and the group are currently in Shanghai, which narrows down our search. One of the posts in particular seems to be about the hotel that the group is staying at (link [here](https://www.instagram.com/p/DSEI-6Wk9kv/)).&#x20;

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Using the first image in this post, we can perform a reverse image search and find the hotel where this landmark is located in.

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

The first result is basically an exact copy of the image in the post. However, the post that the result links to neither contains the image, nor has anything to do with Shanghai, so we keep looking. Adding "Shanghai hotel location" to the search, we get another hit, this time from Trip.com.

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

The website is in Thai, but we can translate it to English. Scrolling down through the reviews, we can indeed confirm that this is the hotel we're looking for (link [here](https://sg.trip.com/hotels/shanghai-hotel-detail-444194/atour-hotel-nanjing-east-road-shanghai-on-the-bund/review.html)).

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

The description of the challenge gives us the flag format, so we just follow that and we can claim our points

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

## Creator's Notes

This was one of the more interesting OSINT challenges that I've made. Originally, I expected this to be very easy, since I managed to find the account by just searching "YBN intern" on Instagram. However, I didn't take into account the fact that the target account was on the same device that I had done the search on, which significantly skewed my search results.&#x20;

Many players had a hard time searching for this account, due to the fact that the word "YBN" only appears in the title, and is not part of the username. As such, I gave the hint that the word "intern" was part of the username, making it much easier to find the account.

Funnily enough, one of the posts was apparently reposted. I'm not sure if this was a participant, or a random account that just so happen to stumble across this by sheer chance.&#x20;

<figure><img src="../../.gitbook/assets/image (21).png" alt="" width="370"><figcaption></figcaption></figure>
