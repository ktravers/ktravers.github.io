---
layout: post
title: Adventures in DNS Configuration
---

###Or How I Stopped Worrying, and Learned to Love [GitHub Support](https://github.com/contact)

One of the main tenants of the [The Flatiron School](http://flatironschool.com) is "Always be a beginner,"* which has been pretty easy for me to embrace so far, since I am the noobiest of coding noobs right now. Pretty much every day, I'm reminded of my beginner-dom, even in realms I thought I'd more or less mastered.

Case in point, this very blog. 

Day one, we were asked to set up technical blogs, which we'd use to write about all the cool new concepts we're learning. I was pretty excited to dive right in, especially since I'd just built my own portfolio site at my newly-purchased custom domain, [kate-travers.com](www.kate-travers.com). When I found out [GitHub](https://github.com) offered free hosting through [GitHub pages](https://pages.github.com/), I about flipped. This was gonna be great. I could build out my new site with a SEO-friendly blog feature, perfect my git skillz and gain all kinds of Github coder cred in one fell swoop. I was a [subdomain and CNAME record](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/) away from [blogging like hacker](http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html). 

Now, this was not my first time at the DNS-configuration rodeo. I'd set up probably +10 domains over the past year, for myself, my friends, even a couple clients. I felt pretty comfortable navigating the GoDaddy's and Domain.com's of the world.

Yeah...you know where this is going. Cue the [klaxon](https://soundcloud.com/shockedandawed/john-hodgmans-klaxon).

The night before I was to present my first blog post to the class, I ran through [Github's clear and thorough instructions](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/), first adding a CNAME record to the master branch of my github pages repo, then going into Domain.com's DNS admin - whom I'd purchased the domain through - and adding a CNAME record there, pointing to my gh-pages site. In the past, I'd seen DNS settings update in minutes, so I was a little concerned when, hours later, my blog still wasn't appearing at its new address. That concern turned to panic when minutes before presentation time, my blog was decidedly not configured.

Shame faced and totally beginner'd, I pulled up my repo and presented the [raw file](https://github.com/ktravers/ktravers.github.io/blob/master/_posts/2015-02-16-refash-your-bash.md) to the class.  

![Strangers with Candy - Shame]({{ site.baseurl }}/assets/shame.png "Jerri Blank, Strangers With Candy - Shame")

After the presentation, I poured over my repo and domain.com's DNS console, trying to figure out where I'd gone wrong. Nothing. No clues. I couldn't see a thing out of place. But that didn't stop me from spending the next few days deleting and re-adding CNAME records, waiting 24hrs, only to have it keep failing, again and again. I read through every [StackOverflow post](http://stackoverflow.com/search?q=github+pages+custom+domain), hoping some other poor noob could save me, and my noob-dom could remain a secret. I'm resourceful. I'd fix this myself.

A week went by, and my resourcefulness had gotten me nowhere. That sad 404 Not Found page kept staring me down, crushing my hacker blogging dreams.

![Homestar Runner 404 Page]({{ site.baseurl }}/assets/homestar404.png "HomestarRunner 404")

Finally, I did what I should have done from the beginning - I contacted Github's support team. An hour later, a nice fellow named Steven! (who I'm 99% sure is not a robot) sent me a polite email, asking why I wasn't using domain.com's default nameservers.   

![Steven's Email]({{ site.baseurl }}/assets/steven.png "Email from Steven!")

Boom. It clicked. Because domain.com isn't hosting my website. I bought my domain name from them, but I bought domain hosting from a nice little outfit called [Web Hosting for Students](http://webhostingforstudents.com/) ($25/year for [Treehouse](http://teamtreehouse.com/) students - I highly recommend it). I could add as many CNAME records to domain.com as I wanted; it'd never redirect my subdomain, because domain.com is my [DNS Registrar](http://en.wikipedia.org/wiki/Domain_name_registrar), not my [DNS Host/Provider](http://en.wikipedia.org/wiki/DNS_hosting_service).

In one email, Steven! had saved the day. A quick visit to my actual DNS provider's admin console, and my blog was in business. 

One email, one lesson from day one at Flatiron - all these first steps, these _beginnings_, are proving to be the most powerful. I'm hoping that's a lesson that'll stay with me 'til the end.


##So what have we learned from this episode?

+ **DNS Registrar != DNS Provider**. If you bought your domain from one company (like domain.com) and your hosting from another company (like studentwebhosting.com), then your DNS Provider is the company providing _hosting._  

+ **[GitHub pages](https://pages.github.com/) are awesome.** Free hosting, amazing support team, what's not to love?  

+ **[GitHub's Support team](https://contact.github.com/) rules.** Especially Steven!.  

+ **Always be a beginner.**  Don't spend hours stubbornly circling a problem. Reach out to another human being. You'll learn something, save hours of your life, and extend your support network - always a good thing.  


_Also in case you're curious, here's the full list of FLATIRON VALUES:_

+ Be a calming force  
+ Create a positive impact  
+ Empower people  
+ Take Risks   
+ Maintain perspective  
+ Find what to love  
+ Make no little plans  
+ Finish what you start  
+ Always be a beginner  
+ Validate yourself  