---
layout: post
title: How to Work with Developers
---

Presented to [Venture for America 2017 Fellows](http://ventureforamerica.org/life-as-a-vfa-fellow/train/)

## Slide deck

<iframe src="https://docs.google.com/presentation/d/1fH8vTFRDtDcIu1-MWyFSHfRpiDr4nqe_9KRPhOMN6kI/embed?start=false&loop=false&delayms=3000" frameborder="0" width="560" height="344" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

## Text (no video available, unfortunately)

Today, I'll be sharing my (completely unbiased) perspective on how to work with developers. My goal for this talk is for everyone to walk out today confident they can work effectively with their development team, thanks to some tools and strategies we'll review together.

### Developers: They're Just Like Us... or are they?

So why is this talk necessary? Are developers really so different from your average co-worker?

**Yes.** Yes, we absolutely are.

Take Flatiron School's engineering team, for example. Here's a picture of us from last summer, happily hacking away in the park outside our office. Why are we in a park? Well, this was moments after our building was evacuated during a fire at about 5:30pm in the afternoon. When the alarms went off, the rest of the office did what you'd expect: went home for the day.

Not us. Without any pre-planning or conversation, every single one of our engineers picked up their laptops, filed outside, sat down on the nearest benches, and immediately went back to work. Like I said, we're a little different.

### Why Bridge the Divide?

As a bootcamp graduate, I didn't begin my career as a programmer. I’ve worked in a number of different fields - both in- and outside- tech - and in every workplace, there’s been a divide between developers and the rest of the company.

Having been on both sides of that divide, I like to think I have a uniquely well-informed perspective, and as far as I can tell, the problems are almost exclusively due to miscommunication.

That’s a good thing, because it means that with a little empathy from both sides, we can bridge the divide.

Succeeding here is in everyone's best interest. It's good for your business to have a happy, productive office. But it's also good on an individual level. Being able to communicate with the “other side” is one of the most valuable skillsets you can bring to the table.

- An engineer who’s a good communicator is perceived as better than an engineer with more technical skills but poorer communication skills.
- Likewise, a non-technical person who can “speak developer” is perceived as more valuable, because engineers are more likely to get stuff done for them.

So what are some of the obstacles we're up against? We'll look at two of the biggest: vocabulary and misconceptions.

### Vocabulary

#### Developer, engineer, programmer

What's the difference between these terms? Effectively nothing. These terms all describe people who code. There's no need to make it any more complicated than that.

For anyone interested in the technical distinction, check out this [blog post from Alan Skorkin](http://www.skorks.com/2010/03/the-difference-between-a-developer-a-programmer-and-a-computer-scientist/).

#### Feature Development

What does the engineering workflow look like at most companies? Most startups will do some version of the agile scrum process.

- Product manager oversees feature ideation, manages backlog.
- Product manager identifies discrete set of stories/tickets for sprint and assigns tickets to developers.
- Developers work through stories, with daily standups and feedback from product manager.
- Engineering and product work with QA to fully test before launch
- Launch / deliver to stakeholders
- Repeat

That's the ideal flow. See the “real” picture: [http://www.projectcartoon.com/cartoon/2](http://www.projectcartoon.com/cartoon/2)


#### Stories / Tickets

During a sprint, engineering work is broken down into stories (which you may also hear referred to as "tickets"). Each story represents a discrete task or unit of work.

Tasks are typically assigned to an individual developer.

At Flatiron School, we like to manage our project queues using Github Projects (read more [here](https://help.github.com/articles/about-project-boards/) and [here](https://github.com/blog/2272-introducing-projects-for-organizations)).

#### The Stack

You may hear engineers referring to the "stack".

> "What kind of stack are you working with?"
>
> "What's your stack look like?"

Wikipedia defines a software stack as:

> “A set of software subsystems or components needed to create a complete platform such that no additional software is needed to support applications” - [https://en.wikipedia.org/wiki/Solution_stack](https://en.wikipedia.org/wiki/Solution_stack)

Here's a diagram of our stack that supports Learn.co.

Yes, I mainly showed this slide to scare you.

A lot of you will be working at a company that uses something that looks just like this.

If you ever ask an engineer to describe your company’s stack to you, they’ll probably sketch out something like this. Entities and relationships. Abstraction, not exhaustive detail. Think of it like this: you don’t think about all the individual components of an engine. You're just fine thinking about gas going in and force for turning the wheels coming out.

So don’t sweat having a perfect grasp on this stuff. It's complicated, even to most engineers.

#### Backend vs. Frontend vs. Full Stack

You'll also hear engineers talking about what part of the stack they work in.

> "We're looking for a front-end specialist."
>
> "She's a backend engineer."
>
> "We expect all our engineers to be fullstack engineers."

These terms loosely refer to what part of the codebase the engineer spends most of their time in.

- Backend deals with data handling.
- Frontend handles UI/UX (User Interface, User Experience).
- Fullstack can do a little bit of everything.

Keep in mind the divisions between these categories are fluid / fuzzy. Few engineers are purely one or the other. Most work across multiple parts of the stack, mostly be necessity.

Read more on the differences [here](http://blog.udacity.com/2014/12/front-end-vs-back-end-vs-full-stack-web-developers.html).

#### API

API stands for Application Program Interface.

APIs define an interface that developers can use to request and receive data from a data source.

Flatiron School's founder, Avi Flombaum, likes to use the restaurant analogy to explain APIs. Say I go to a restaurant and I really want a grilled cheese sandwich. I'm not just going to walk into the kitchen and start grabbing sandwich ingredients. Instead, there's a protocol. I ask the waiter for a menu, and make my selection from the menu's pre-defined choices. My request goes in, then the waiter brings me back a grilled cheese sandwich.

Think of that menu as your API. It has predefined choices set by the restaurant that allow you to request and receive food.

A public API works the same way. For example, if I want to put a Twitter feed on my website, I can use [Twitter's API](https://dev.twitter.com/overview/api) to request tweet data, following the rules they've defined and documented.

Read more on the [HandsConnect blog](https://www.handsonconnect.org/blog/2016/8/17/what-is-an-api-and-why-should-i-care).


#### The :shipit: Squirrel

This little cutie started as an inside joke at [Github](https://github.com), but its adorableness quickly permeated the industry. When your code has been approved to ship, your teammates tag it with the :shipit: squirrel, the universal mascot of pushing code.

Read the full lore [here](https://www.quora.com/GitHub-What-is-the-significance-of-the-Ship-It-squirrel) and [here](https://github.com/blog/1271-how-we-ship-github-for-windows).


### Misconceptions

#### True or false? "Engineers... do not like speaking with people. Coding all day is good fun, talking with people is torture."

False.

Engineers are just as social as your average co-worker.

We’re just very protective of our precious (and limited) attention span.

1. Choose the appropriate communication channel for the situation.
  - Emergency: cannot wait, equivalent to pulling someone out of a meeting
  - Urgent: within the hour
  - General: within the next few days / weeks

2. Most of the time, talk to a manager first. Trust them to triage appropriately.


#### True or false? When you ask for something from an engineer, don't get too detailed. They're the experts, so let them decide how to do it.

False.

Engineers are detail-oriented problem solvers.

Ambiguity slows us down; the more blanks we have to fill in, the more likely we’ll end up with a final product that doesn’t make anyone happy.

Don’t hide things from developers; if you think feature requirements will change soon or eventually, disclose that up front.

Here's a [true story](https://www.salesforce.com/video/296975/) about the importance of being precise. A Salesforce product manager asked that “when an account was updated, shoot the owner an email.”  The developer said ok. Then the first email the account owner received read “TRUE”.

Remember to clearly define your desired **outcome**, but not the means of getting there. For example, often times a stakeholder will ask for data in a specific format - like CSV - which they then take and upload into Google Sheets and spend hours making pivot tables, etc. An easier route would have been to just tell the developer that they need to know X from this data. The developer might know an easier, faster means of analyzing the data (using tools like [Periscope](periscopedata.com), for example).

#### True or false? Engineers love details and hate meetings, so don’t bring them into a project until you’ve mapped everything out completely in advance.

False. Bring engineers in early.

> “There’s a tendency in many companies to “insulate” the development teams from “the business” — usually in the name of trying to reduce randomization and to ensure execution overall. This is a very sideways way of thinking, which usually results in expecting people to execute without context — without understanding the vision, the strategy, the tactics, or especially the customer. And therein lies the problem — people are motivated most when they share a vision of what the future might be, and can see themselves and their contributions in that picture.” [[source]](https://community.uservoice.com/blog/work-effectively-engineers/)

> “Engineers are tremendous assets in brainstorming sessions and when reviewing initial designs” [[source]](https://www.nczonline.net/blog/2012/06/12/the-care-and-feeding-of-software-engineers-or-why-engineers-are-grumpy/)

> “You should consult engineers early in the process, when the product is still being defined. If you don't, you risk both charting an impossible course and alienating your engineer.” [[source]](http://www.peachpit.com/articles/article.aspx?p=99803)

### Exercises

Let's practice what we've learned so far. All examples are credit [Spencer Rogers](https://github.com/spencer1248).

#### Conversation 1: Project Manager to Engineer

> **Project Manager:**
> How’s the password reset feature going?
>
> **Developer:**
> I started looking into the Postmark API and installed their client library, but I started running into some issues in my development environment because of an outdated library we’re using for image handling.

**Rough translation:**  
_I am working._

**How could this make the project manager feel?**  
_Confused, agitated, stone-walled, impatient, questioning the common sense of the developer_

**Is this good communication?**  
_No._

**What was the meaningful information?**  
_I need an update on your progress, so I can assess whether to assign more or less resources._

**What would be useful to have said instead?**  
_I hit a roadblock, which set me back by ~1 day._

**How could the other party respond to invite a better answer?**  
_How does this affect the original estimate? Do you need anything from me to get past this?_

**How could the initial question have been asked better?**  
_I’m working on assignments for next week. Can you give me a quick status update on the password reset feature?_

#### Conversation 2: Marketing Manager to Engineer

> **Marketing manager to developer:**
> Can you please build us something to address the sign-up conversion rate by the end of the day?

**Rough translation:**  
_Can you do something you don’t know how to do in an arbitrary time frame that may or may not be realistic?_

**Is this good communication?**  
_No._

**How does this make the developer feel?**  
_Overwhelmed, unsupported, disrespected, cog-in-machine, irritated that question is so ignorant of their capabilities, reality, etc._

**What was the meaningful information?**  
_We need something built to solve a business problem._

**What would be useful to have said instead?**  
_Not a lot of people are signing up for the new service. Do you know how we can fix it? No? Well let’s figure something out that you can build within our deadline._

_Not a lot of people are signing up for the new service. Do you know how we can fix it? Yes? That’s great. How long would something like that take to build? (then potentially) That’s too long, is there a simpler or interim solution we could use?_

**How could the other party respond to invite a better answer?**  
_I can try my best, but I’ve never had to do that before and I don’t have any idea how long it will take or whether it will be effective._


#### Conversation 3: Engineer to CEO

> **Developer:**
> I just used a really cool algorithm to guess similar words using something called “Levenshtein distance”. The data consistency problem should be fixed by EOD.
>
> **CEO:**
> Ok, but what about the landing page update?

**Developer Rough translation:**  
_I just did something cool to fix the problem and deliver it on time._

**CEO Rough translation:**  
_I expect you to fix problems like this, what about the stuff that’s actually going to move this business forward?_

**Is this good communication?**  
_Nope._

**How does the developer feel?**  
_Proud, unhappy that their work wasn't appreciated, rushed._

**How does the CEO feel?**  
_Agitated that something that set the business back is being celebrated before something that is completing the CEO’s vision of the company is finished._

**Two sides:**  

_Developers spend 99% of their day working on stuff that many people can’t see or appreciate. It can be frustrating, especially when they’ve built something they’re proud of and it’s all they’ve thought about for weeks._

_Some developers make themselves less efficient simply in order to manage the expectations of the people waiting on their product. It is tough to gauge if you don’t have a window that you can trust._

_There’s also many ways to be a “good” developer. At a small startup, a good metric is whether the developer is producing value for the company, which can be difficult to measure. Many of the ways a developer produces value are not visible to a lot of the company._

_CEOs ultimately want their company to be successful. As a developer (or anyone), you should do your best to understand the high level goals of the company._

_This doesn’t mean that they’re going to be mean by default. If you had 2 companies, both producing the exact same revenue, but one had a healthy culture and one had an unhealthy culture, which one would you choose as a CEO?_

_It might be better to find someone else to tell about your cool algorithm, though._


#### Conversation 4: Engineer to Product Manager

> **Developer to product manager:**
> I’m working on the search feature and noticed that some of these filtering options don’t make sense together. What if we did it like this instead?

**Rough translation:**  
_I was working on this but my common sense kicked in and realized it didn’t make sense. I have an idea that would fix it though._

**Is this good communication?**  
_Yes._

**What was the meaningful information?**  
_Everything._

**What would be useful to have said instead?**  
_Maybe an estimate for the proposal._

**How could this have gone wrong?**  
_If the dev had changed stuff without talking to someone. It doesn’t mean they have to give up total ownership of the product, but everyone should be on the same page. A lot of people have probably been thinking about this feature._

_The best course of action here isn’t the best course of action everywhere. Have an understanding of how these things should work._

### Takeaways: Things You Can Do Right Away For Great Good

1. Take an engineer out for coffee :)
  - Get their perspective on this talk - what’s their advice for working well with developers?
  - Ask them about what they’re working on
  - Ask about non-work stuff too
  - In general: spend time with people from other departments - game nights, lunches, coffee
2. Attend open engineering events (demos, retros, code talks)
  - Get familiar with the team and what they’re working on
  - Get comfortable with shared vocabulary, company terminology
3. Get into the product
  - Look for opportunities to work with developers (doesn’t have to be on a development project; could be event organizing, etc)
  - Volunteer to help out on other teams. Some personal examples:
    - helping with QA / user testing
    - hosting meetup events (and helping with set up / tear down)
    - writing blog posts
  - user outreach

Last but not least, let developers know when you appreciate their work. Thank you!


### Resources: Articles

1. [Krzysztof Rakowski - How To Communicate Effectively In IT Projects](https://www.smashingmagazine.com/2014/06/communicating-effectively-in-projects/)  
2. [Julie Zhuo - How to Work with Engineers](https://medium.com/the-year-of-the-looking-glass/how-to-work-with-engineers-a3163ff1eced)  
3. [Nicholas Zakas - The care and feeding of software engineers](https://www.nczonline.net/blog/2012/06/12/the-care-and-feeding-of-software-engineers-or-why-engineers-are-grumpy/)  
4. [Cliff Gilley - How to Work Effectively With Engineers](https://community.uservoice.com/blog/work-effectively-engineers/)  
5. [June Cohen - How to Work with Engineers on a Web Development Project](http://www.peachpit.com/articles/article.aspx?p=99803)  
6. [Stella Garber - 5 Best Practices for Working with Developers](https://www.forbes.com/sites/stellafayman/2013/03/14/5-best-practices-for-working-with-developers/#5274f3153ef1)

### Resources: Videos

1. [Laura Klein - Building Happy Product Teams like Heist Teams](http://www.mindtheproduct.com/2016/07/building-happy-product-teams-like-heist-teams-laura-klein/)  
2. [Ryan Hughes - Bridging the Gap between Designers and Developers](https://www.youtube.com/watch?v=mweTM1hsn84)  
3. [Salesforce Case Study - How Admins And Developers Can Collaborate](https://www.salesforce.com/video/296975/)  
4. [​Ron Lichty - How to Get Your Development Team to Love You](http://www.mindtheproduct.com/2016/04/product-owners-how-to-get-your-development-team-to-love-you/)
