---
layout: post
title: Notes on Accelerate
tags: [book report]
---

I'm reading ["Accelerate" by Jez Humble, Gene Kim, and Nicole Forsgren](https://learning.oreilly.com/library/view/accelerate/9781457191435/) along with some folks at work (including author Nicole Forsgren, Ph.D!)

Here's my notes on each chapter.

## Part One: What We Found

### Chapter 1 - Accelerate

- "...our research shows that none of the following often-cited factors predicted performance: ...whether a change approval board (CAB) is implemented"
- Summary of leading indicators:
  - Continuous deployment: "46 times more frequent code deployments" (aka "tempo")
  - Well-scoped PRs: "440 times faster lead time from commit to deploy"
  - Easy rollbacks: "170 times faster mean time to recover from downtime" (aka "stability")
  - High confidence deploys: "5 times lower change failure rate (1/5 as likely for a change to fail)"

#### Questions

- The book mentions that a "change approval board" isn't an indicator of success, so is it an indicator of something else? A warning sign / red flag?


### Chapter 2 - Measuring Performance

TLDR: This chapter covers the four measures of delivery performance derived by the authors from their survey results:

1. Delivery Lead Time: time it takes for work to be implemented, tested, and shipped
1. Deployment Frequency: how often code is deployed to production
1. Time to restore service: time it takes to recover from an incident
1. Change fail rate: % of changes to production that result require remediation

These four measures fall into two buckets:  
1. Tempo: delivery lead time and deployment frequency
1. Stability: time to restore and change fail rate

In general, high performing organizations demonstrate both high tempo and high stability:  
- Short lead times
- High deploy frequency
- Low time to restore
- Low change fail rate

And now the full summary...

#### The Flaws In Previous Attempts To Measure Performance
_In which we learn Apple tried measuring lines of code for a hot second_

Measuring the performance of software development teams is hard; just ask the folks at Velocity (or our Insights team 😎). But in general, your approach is flawed if you're focusing on 1) outputs over outcomes and 2) local over global (individual vs org goals).

Three common examples of flawed metrics:  
1. Lines of code. Whichever direction you go (maximizing vs minimizing), you end up with difficult to maintain code. If you didn't read the Apple anecdote from the footnotes, it's pretty cute.
1. Velocity. Unreliable because it's context-specific, so same benchmarks can't be applied to all teams. It's easy to game by inflating estimates, and it can disincentivize cross-team collaboration (helping another team boosts their velocity but slows yours down, making you look bad by comparison).
1. Utilization. You need some slack to account for surprises (and there's always surprises). Plus if every team is stretched thin, they won't be performing at their best and are at risk for burnout.

#### Measuring Software Delivery Performance
_A peak behind the research curtain_

A better approach to measuring performance is to focus on 1) outcomes over output and 2) global outcome over individual or team. That way you're disincentivizing cross-team rivalries and incentivizing collaboration + work that advances system-level goals.

Based on their research, the authors selected four measures that meet these criteria.

1. Delivery Lead Time  
  - Key element of Lean theory (sidenote: O'Reilly has lots of resources on Lean)
  - "Lead time" can be divided into two phases for product development: 1) product design and 2) product delivery, which is less variable, so easier to measure.
  - Important for shipping to learn + quick course correction

2. Deployment Frequency  
  - Proxy for "batch size", which applies to manufacturing or tangible products
  - Goes hand-in-hand with lead time in improving cycle time, accelerating feedback, etc.
  - Selected because it's easy to measure and tends to have low variability

Together delivery lead time and deployment frequency measure software delivery performance tempo or pace. Often folks think improving tempo means sacrificing stability (more on that later). The next two measures both fall under "stability" or health of your system.

3. Mean Time to Recover (MTTR)  
  - Exactly what it sounds like: how long does it take to restore service after an incident?
  - Accepts that failure is inevitable; some teams just handle it better than others

4. Change Fail Percentage  
  - What percentage of changes require remediation? (hotfix, rollback, etc).
  - aka "Percent Complete and Accurate" in Lean theory (another key quality metric)

After surveying participants on the four measures above, the authors used cluster analysis to group the results into performance categories: high, medium, and low performers.

The authors were surprised that high performers didn't have trade stability for tempo (or vice versa). Instead, high performers were better across all four measurements, which matches up with what Agile and Lean methodologies promise.

#### The Impact of Delivery Performance On Organizational Performance
_Get Lean and $Profit_

Why measure delivery performance? Turns out it's highly correlated with organizational performance (profitability, market share, productivity, ROI). It also predicts "non-commercial goals" like customer satisfaction, quantity and quality of services.

For these reasons, the authors advocate for keeping strategic software development in-house. Software that isn't related to one of your "core competencies" should be bought instead of built (ex. payroll systems).

#### Driving Change
_Now the hard part..._

Now that we know what "good" software development performance looks like, we can start making data-driven decisions and what and how to change. However the authors caution us to be careful. Taking a scientific approach to improving performance works well in "learning cultures", but not so well in "pathological and bureaucratic organizational cultures".

#### Questions

- Which of the four performance metrics does GitHub have the most room to grow?
- If you had to pick one of the four performance metrics for GitHub to invest time in improving, which one would you choose?
- What is GitHub's "Change Fail Percentage"? Do we track this anywhere (ex. in Sentry, Datadog)?
- The authors call out one surprise in the yearly trends (medium performers had a worse change fail rate than low performers in 2016). Anything else that surprised you in those yearly numbers?
  - Why did low performers have a huge spike in mean time to recovery in 2017?
  - Why so much variability year-to-year in change lead rate for low performers?
  - Why the big spike in deploy frequency for high performers in 2016?
- From the last section, Driving Change: "Do change management boards actually improve delivery performance? (Spoiler alert: they do not; they are negatively correlated with tempo and stability.)" Is our Rollout Advisory Board a "change management board"? If yes, now that it's been a place for a little while, what have we learned? Can we apply any lessons from this book yet?
- Thinking about our Insights product:
  - How does Insights measure tempo and stability? Are we using the same measures discussed in this chapter? Any alternative or additional measures?
  - Have any of the performance measures discussed in this chapter proven difficult to capture or convey?

#### Discussion


### Chapter 3 - Measuring and Changing Culture

#### Questions

#### Discussion

### Chapter 4 - Technical Practices

#### Questions

#### Discussion

### Chapter 5 - Architecture

#### Questions

#### Discussion

### Chapter 6 - Integrating Infosec into the Delivery Lifecycle

#### Questions

#### Discussion

### Chapter 7 - Management Practices for Software

#### Questions

#### Discussion

### Chapter 8 - Product Development

#### Questions

#### Discussion

### Chapter 9 - Making Work Sustainable

#### Questions

#### Discussion

### Chapter 10 - Employee Satisfaction, Identity, and Engagement

#### Questions

#### Discussion

### Chapter 11 - Leaders and Managers

#### Questions

#### Discussion

## Part Two: The Research

### Chapter 12 - The Science Behind This Book

#### Questions

#### Discussion

### Chapter 13 - Introduction to Psychometrics

#### Questions

#### Discussion

### Chapter 14 - Why Use a Survey

#### Questions

#### Discussion

### Chapter 15 - The Data for the Project

#### Questions

#### Discussion

## Part Three: Transformation

### Chapter 16 - High-Performance Leadership and Management

#### Questions

#### Discussion