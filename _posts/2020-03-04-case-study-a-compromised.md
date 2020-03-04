---
title: "Case Study: A Compromised WordPress Site in the Wild"
date: 2020-03-04T00:00:00-00:01
categories:
  - blog
tags:
  - blog
  - write-up
share: true
permalink: /case-study-a-compromised-wordpress-site/
---

Recently, I've been making arrangements with family to fly in and visit the Windy City sometime this summer. I was looking for interesting events and came across a taco and tequila event hosted on a website by the name of Chicago Events which, frankly, sounded pretty fun.

What *wasn't* fun was clicking the link to see the desired event page load briefly, then abruptly render an obviously suspicious Adobe Flash Player update message. I know, I know, that trick has been around for years. The odd part? I wasn't redirected, nor was this a pop-up - I was still on the legitimate page I'd requested. It was time to look under the hood.

<figure>
  <img src="{{ '/assets/images/2020-03-03/wordpress1.png' | relative_url }}" alt="Fake Adobe Flash Player warning">
</figure>

I pulled up Safari's dev inspection tools and started working on determining what had occurred. Obviously, the site was compromised. But how?

The first thing that stuck out to me was that the website was built using the [notoriously insecure WordPress](https://wordpress.org/support/article/faq-my-site-was-hacked/), with indicators like the path */wp-content/*, which serves themes and plug-ins.

<figure>
  <img src="{{ '/assets/images/2020-03-03/wordpress2.png' | relative_url }}" alt="HTML excerpt of some content in /wp-content/">
</figure>

At this point, I had a hunch. I'd previously read on the subject of the truly cowboy-style wild west that is WordPress plug-ins and themes, and kept this in mind for what was (hopefully) going to come.

Back in the dev inspection tools, poking around a bit more revealed a dubious frame being served from an oddly named website - dabeolu[.]art.

<figure>
  <img src="{{ '/assets/images/2020-03-03/wordpress3.png' | relative_url }}" alt="Dev inspection tools showing dabeolu.art frame">
</figure>

I ran it through DomainBigData to get 85.193.126[.]150. A host in Sweden. WHOIS information led me to  [njal.la](https://njal.la/), or Njalla, a domain name registration service that is privacy-oriented and, apparently, based out of an island in the Caribbean.

Interestingly, at the time of writing, their website was down.

<figure>
  <img src="{{ '/assets/images/2020-03-03/wordpress4.png' | relative_url }}" alt="Simple, temporary webpage showing they are offline and recovering">
</figure>

-

Back on topic - I decided to keep inspecting resources found in the dabeolu[.]art frame. I saw there were two JavaScript files also associated and dove in. One of the two, *main.js*, was simply this:

<figure>
  <img src="{{ '/assets/images/2020-03-03/wordpress5.png' | relative_url }}" alt="Code block assigning download_link variable to malicious BitBucket file">
</figure>

Attempting to click the Flash Player update link would thus retrieve a malicious HTA from the attacker's BitBucket account. I nabbed it and popped it open with Sublime Text to see what its goal was. Waiting for me was some obfuscated VBS.

<figure>
  <img src="{{ '/assets/images/2020-03-03/wordpress6.png' | relative_url }}" alt="Obfuscated VBS in Sublime Text editor window">
</figure>

Without making this write-up *much* more lengthy and going down the reverse engineering/de-obfuscation rabbit hole (although that may warrant its own future post), I ended up just uploaded the file to one of my favorite dynamic analysis tools to see what was happening. For those interested, the full details are available [here](https://www.hybrid-analysis.com/sample/36c8c44b9baf743479d1a610af49598daf1c2154c9c938989653917159d64fac/5e56f418d0f56725245348dd).

I was still convinced that this was the result of a compromised plug-in or theme, but needed more information. This time I queried everything for dabeolu[.]art and got a solitary hit at the end of a jQuery library being served by our **victim's site(!)** at the path */wp-includes/js/jquery/jquery.js?ver=1.12.4-wp*:

<figure>
  <img src="{{ '/assets/images/2020-03-03/wordpress7.png' | relative_url }}" alt="dabeolu.art dev inspection tool search results">
</figure>

A subsequent series of rapid Google searches led me to endless Stack Overflow threads where poor WP developers and admins had logged in to discover their sites had gone rogue. Corrective action seemed easy enough, detailing fixes such as restoring from a backup or grabbing a fresh copy of jQuery from [the official website](https://jquery.com/download/). It appeared common, and remediation seemed pretty straightforward, so both were positives.

So here we are. In summary: a WordPress site had been compromised with research indicating a vulnerable plug-in or theme, and we were now dealing with a bad actor who had managed to set up a clever persistent XSS attack by using cookies and a jQuery file to deliver the payload via carefully crafted conditional code. Quite the mouthful.

But wait… where did I figure that last part out? Savvy readers (what readership?!) will notice the code block to the right of the screenshot above has some interesting cookie talk going on, and no, we're not debating chocolate chip versus oatmeal. I think this calls for a part 2, where I hope to do some deeper analysis of the attack methods used and find out what exactly that HTA does.

This is also available on [Medium](https://medium.com/@safekeepr/case-study-a-compromised-wordpress-site-in-the-wild-39295e613b6) as my first write-up for them.

Stay tuned!