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

### Chapter 3 - Measuring and Changing Culture

- Many different definitions of culture:
  - National (what country you belong to)
  - Organizational
    - Basic assumptions
      - Formed over time based on visible interactions
      - More implicit, hard to articulate
    - Values
      - Collective values and norms
      - Easily articulated, discussed, debated
      - Lens through which to view and interpret interactions
    - Artifacts
      - Written statements
      - Heroes
      - Rituals
- Westrum's typology
  - Three types:
    - 1. Pathological
      - Power-oriented
      - Focused on individual
      - Low cooperation
      - Messengers punished
      - Failure punished
      - Driven by fear, threats
      - Information is hoarded, withheld, or distorted for political gain
    - 2. Bureaucratic
      - Rule-oriented
      - Focused on teams/departments
      - Modest cooperation
      - Messengers ignored
      - Failure "leads to justice" (?)
      - Competitive, lots of fences between departments
    - 3. Generative
      - Performance-oriented
      - Focused on company mission, goals
      - High cooperation
      - Messengers trained
      - Failure leads to questions, learning
  - Type predicts information flow
  - Good info flow is critical to safe + effective operations
  - What is good information?
    - Answers questions the receive needs answered
    - Timely
    - Presented in a way that the receiver can use it
  - Better information -> better decision making
  - Also easier to reverse decisions that don't work out (very important)
  - Research shows Westrum's theory predicts safety outcomes, software delivery, and organizational performance

#### Questions

- So... what is culture?
  - Culture is the assumptions, values, and rituals of a company
  - Culture predicts info flow, and info flow predicts decision making, and decision making predicts success

#### Discussion

- Appreciate the Likert scale survey w/ questions based on Westrum's model.
- Advice: start by changing behavior, not how people think

### Chapter 4 - Technical Practices

- Discussing XP vs Scrum
  - Will we touch on embedding stakeholders in project team?
- Research shows technical practices are just as important as management and team practices
- Discovered that continuous delivery practices have an effect on team burnout and deployment pain
- Five key principles of Continuous Delivery:
  1. **Build quality in**
    - Eliminate need for "inspection" aka QA testing?
  2. **Work in small batches**
    - Shorter feedback cycles
    - Aim to make cost of pushing out individual changes low
  3. **Use computers for repetitive tasks, people for solving tough problems**, aka automate as much of the mundane stuff as possible
    - Free up people for higher order + higher value problem solving
  4. **Relentlessly pursue continuous improvement**
    - Don't be satisfied; always work to improve
    - Improvement is part of everyone's daily work
  5. **Everyone is responsible**
    - Shared risk, shared responsibility, shared reward
    - It's management's responsibility to make the state of system-level outcomes transparent in order to set goals and measure progress
- Foundations of / pre-requisites for continuous delivery:
  - **Comprehensive configuration management**
    - "It should be possible to provision our environments and build, test, and deploy our software in a fully automated fashion purely from information stored in version control"
  - **Continuous integration**
    - Changes are merged frequently
    - Every change triggers a build process tha tincludes running unit tests
    - Failures are fixed immediately
  - **Continuous testing**
    - Automated unit and acceptance tests run against every commit
    - Fast feedback
    - Devs should be able to run all tests locally
    - Work is not "done" until tests are written and passing
- Security should be in process rather than downstream
  - What does this mean concretely in practice?
- Good question: research shows CD increases developer happiness, etc., but does it increase quality? Yes.
  - Lower levels of unplanned work (!!)
  - Lower levels of rework (!!)
- Version control
  - "...keeping system and application configuration in version control was more highly correlated with software delivery performance than keeping application code in version control". Good news for HashiCorp.
- High quality tests:
  - High confidence application works when tests pass
  - High confidence test failures indicate an actual defect
  - How to deal with low quality tests:
    - Separate quarantine suite that is run independently
    - "just delete them" <3
  - On QA:
    - "having automated tests primarily created and maintained either by QA or an outsourced party is not correlated with IT performance"

#### Questions

- What is "comprehensive configuration management"? (mentioned in first paragraph in context of "Continuous Delivery", Humble and Farley 2010)
- By "building quality in" to continuous delivery, does that eliminate or decrease the need for quality assurance testing?
  - PHEW: "None of this means that we should be getting rid of testers."
- What are examples of bringing security in process with software delivery?
  - "infosec personnel provided feedback at every step of the software delivery lifecycle, from design through demos to helping with test automation"
- New Work vs. Unplanned Work graphs also show high performers having less time in meetings than low performers. Will that be discussed further later?
- Is the "GitHub Flow" workflow mentioned here out of date? Do we still stand by it? (Note: `https://guides.github.com/introduction/flow/` says it was "Last updated Nov 30, 2017")

#### Discussion

- Excellent, actionable advice: "If you want to improve your culture, implementing CD practices will help"
- What does "building quality in from the beginning" mean? Concrete examples?
  - Discuss and address security, observability, performance up front.
  - Include security in design process, automation for security in development, demos, etc.
  - Include these "quality" checks at key touch points: ideation, planning, development, demo'ing, testing
- "Grandmas are always right" :)
- "Exploratory testing", not QA testing. Exploratory testing is very different mindset from typical developer testing. And if you don't have QA people doing exploratory testing, then your customers end up doing the "testing".

### Chapter 5 - Architecture

- Architecture can be a "significant barrier to increasing...tempo and stability". Oh, don't I know it.
- Good to know that "high performance is possible" if everything is loosely coupled (both teams and architecture)
- Research on types of systems
  - Types of systems:
    - Greenfield (unreleased)
    - Engagement (used directly by end users)
    - Record (storing business-critical info, high data consistency and integrity)
    - Custom 3rd-party software
    - Custom in-house software
    - Off-the-shelf 3rd-party software
    - Embedded software on hardware device
    - Software with user-installed component (including mobile apps)
    - Non-mainframe software on 3rd-party servers
    - Non-mainframe software on on-prem servers
    - Mainframe software
  - Learnings
    - Low performers:
      - Custom 3rd-party software
      - Mainframe systems
    - Otherwise no significant correlation between performance and type of system
      - Expected packaged software, systems of record, embedded systems to perform worse, but wasn't the case
      - Expected engagement, greenfield systems to perform better, but no
    - High performing architectural characteristics:
      - Testability: Tests do not require integrated environment
      - Deployability: Application can be deployed independently of apps/services it depends on
      - Architecture and teams are loosely coupled:
        - Large scale changes can be made to the design of the system without permission of someone outside the team
        - Large scale changes can be made to the design of the system without creating work for other teams
        - Work can be completed with out collaborating w/ people outside team
        - Deploy and release on demand
        - Test on demand w/o an integrated test environment
        - Deploy during regular business hours without downtime
      - Build security into work
        - "information security teams make preapproved, easy-to-consume libraries, packages, toolchains, and processes available for developers and IT operations to use in their work"
  - Conclusions:
    - Focus on architectural characteristics, not implementation details
    - Better to bring critical stuff in-house
    - Delivery teams need to be cross-functional
    - Focus on outcomes, not tools
  - Examples:
    - Bounded contexts and APIs for decoupling
    - Test doubles and virtualization for testing in isolation

> "Unfortunately, in real life, many so-called service-oriented architectures don't permit testing and deploying services independently of each other, and thus will not enable teams to achieve higher performance."

- What we should be aiming for (nbd)
  - Goal-oriented generative culture
  - Modular architecture
  - Engineering practices that enable continuous delivery
  - Effective leadership


#### Questions

- In what ways does our architecture mirror the communication structures of our organization? (Conway's Law)
- Let teams choose their own tools... ok, but what happens when you choose the wrong tool?

#### Discussion

- See Conway's Law discussion on breaking up our monolith

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
