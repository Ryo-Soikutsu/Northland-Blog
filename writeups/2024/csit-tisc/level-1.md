---
description: Navigating the Digital Labyrinth
---

# Level 1

## Challenge Description

{% code overflow="wrap" %}
```
The dust has settled since we won the epic battle against PALINDROME one year ago.

Peace returned to cyberspace, but it was short-lived. Two months ago, screens turned deathly blue, and the base went dark. When power returned, a mysterious entity glitched to life on our monitors. No one knows where it came from or what it plans to do.

Amidst the clandestine realm of cyber warfare, intelligence sources have uncovered the presence of a formidable adversary, Vivoxanderith—a digital specter whose footprint spans the darkest corners of the internet. As a skilled cyber operative, you are entrusted with the critical mission of investigating this elusive figure and their network to end their reign of disruption.

Recent breakthroughs have unveiled Vivoxanderith's online persona: vi_vox223. This revelation marks a pivotal advancement in our pursuit, offering a significant lead towards identifying and neutralizing this threat.

Our mission now requires a meticulous investigation into vi_vox223's activities and connections within the cyber underworld. Identifying and tracking Vivoxanderith brings us one crucial step closer to uncovering the source of the attack and restoring stability to our systems. It is up to you, agent!
```
{% endcode %}

Category: OSINT

## Walkthrough

Starting off this challenge, we can see that we're already given a username to work with, "vi\_vox223". Since this seems to be an OSINT challenge, I start off with a Sherlock scan

<figure><img src="../../.gitbook/assets/image (125).png" alt=""><figcaption><p>Honestly I should've seen it coming that this was an Instagram account</p></figcaption></figure>

Navigating to the URL found, we get the following account

<figure><img src="../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

Checking the "AI" stories yield no results, other than a dirty look from an artist friend (who happened to be seated nearby). However, looking through the "Discord" stories was a whole new story

_(continuing the first stage of reconnessiance on my phone because Insta refused to log me in)_

<figure><img src="../../.gitbook/assets/image (127).png" alt="" width="375"><figcaption><p>Looks like our friend here is getting into bot development</p></figcaption></figure>

Reading through the stories, it seems like our friend vi\_vox223 is developing some kind of bot for their own goals. Fortunately, he seems to have given us some valuable tidbits to continue

<figure><img src="../../.gitbook/assets/image (128).png" alt="" width="188"><figcaption><p>Hmm, a secret role?</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (131).png" alt="" width="188"><figcaption><p>How nice of him to give us the ID</p></figcaption></figure>

We got the bot ID as well as a secret role, now all we need to do is to invite the bot to a private server and give ourselves the role. To invite the bot, I used a bot permissions calculator website (link to it [here](https://discordapi.com/permissions.html#0)), and plopped the bot token we got in

<figure><img src="../../.gitbook/assets/image (132).png" alt=""><figcaption><p>Calculating the invite link</p></figcaption></figure>

Pasting it into our browser displays the bot invite prompt, which we invite to a private server

<figure><img src="../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

Running the !help command without the hidden role, we see that it looks very innocent. After adding the "D0PP3L64N63R" role however, the help command now returns a vastly different result

<figure><img src="../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

We can see a couple of file and directory manipulation commands now accessible to us. Checking through some of these files reveals an interesting file to download

<figure><img src="../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

For the uninitiated, an .eml file extension indicates that it is an email file. We can use a desktop application like Microsoft Mail or Outlook to read the contents of the email

<figure><img src="../../.gitbook/assets/image (137).png" alt=""><figcaption><p>God, there so much moreeee</p></figcaption></figure>

Now we have to track this man? Damn, this is a bit of a tedious first challenge

Looking through the email, we see that we are given 3 strings, followed by a hint that they are used as a part of Uber's GPS system. A quick search yields us Uber H3, a custom-made tool for visualizing and exploring geospatial data (to find out more, click [here](https://www.uber.com/en-SG/blog/h3/)). Luckily, the blog linked includes a public demo of the tool, which we can use to paste in the 3 strings.

<figure><img src="../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

Cross-referencing this with Google Maps (we use the area labelled Cimitero di Sermoneta to narrow down the location), we can find the midpoint of the 3 cells

<figure><img src="../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

Our final location is Quercia secolare. But we're not done here yet, so lets continue reading the email

The email also states to contact them through some communications method, that is accessible on a particular organization's LinkedIn profile. Switching to the Posts tab, something immediate catches my eye

<figure><img src="../../.gitbook/assets/image (141).png" alt="" width="302"><figcaption><p>A Telegram bot? Interesting</p></figcaption></figure>

The link takes us to Telegram Web, opening a chat with the bot. We can type any text into the chat, and it will get the definition of the word for us

<figure><img src="../../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

Pasting in the location that we found earlier, we get the flag

<figure><img src="../../.gitbook/assets/image (143).png" alt=""><figcaption><p>(Ignore the previous messages, I was being an idiot and couldnt read)</p></figcaption></figure>
