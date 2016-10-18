---
layout: post
title: CSMess to OOCSS - Refactoring CSS With Object Oriented Design
---

[Presented at Fullstack Toronto Conf. 2016](https://eventmobi.com/fstoco2016/agenda/141857/806561)

## Slide deck

<iframe src="https://docs.google.com/presentation/d/12H2MLnGdaFN2xk-MRVyIRX4j77zQeAHytcUmFcQ6b2Q/embed?start=false&loop=false&delayms=5000" frameborder="0" width="560" height="344" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

## Outline

CSMESS TO OOCSS  
Refactoring CSS With Object Oriented Design

---

### INTRO

Kate Travers // [kate@flatironschool.com](mailto:kate@flatironschool.com) // [@kttravers](https://www.twitter.com/kttravers) // github: [ktravers](https://www.github.com/ktravers)

Web Developer at [The Flatiron School](www.flatironschool.com), NYC

Working on [Learn.co](https://learn.co), platform that powers Flatiron’s online campus

---

CSS is awesome, but at Learn.co, ours definitely wasn’t.

Our CSS was easily the messiest part of our web app - which probably sounds familiar to many web devs out there.

This spring (2016), we budgeted time + resources to overhaul it. I led that project. This is my story.

---

Before moving any further, let's clarify what we mean when we talk about OO design.

OO design is code that is:  

1. Modular - reusable
2. Encapsulated - minimize dependencies
3. Maintainable - esp. important for smaller dev teams

But how does OO apply to stylesheets?

---

Some background: Learn.co is built on a Rails backend.

As Rubyists, we love OO design... except when it came to our CSS. And it's not that we weren't trying. We were just doing it wrong.

Here’s your average stylesheets directory tree:  
- one sheet per view
- within each sheet, styles are namespaced under top level id

This approach is fine for smaller projects, but over time, gets out of hand. Grows bloated + broken, because not actually achieving goals of OO design.

**NOT MODULAR**

No re-use =>  

- lots of code duplication  
- inconsistency in code implementation and visual UI

**NOT ENCAPSULATED**

Improper namespacing =>  

- CSS specificity battles  
- Unpredictable behavior when you change a class name or try to reorganization sheet

**NOT MAINTAINABLE**

Painful to maintain =>  

- Writing new CSS for every view => slows down pace of development  
- Massive bloat over time => slow pageloads  
- Lots of churn => inconsistencies in code and design  
- Not future-proof. Standards change, but painful to update across tons of stylesheets

Good luck trying to onboard into that.

---

We knew our CSS was broken. But what’s the solution?

Enter...

OOCSS (insert snappy Pulp Fiction one-liner here)

---

Once we decided to move to OOCSS, we had another decision to make: use existing OOCSS framework or roll our own?

OOCSS is not a new thing. There are lots of frameworks out there:  

- Twitter Bootstrap
- Yahoo Pure
- Zurb Foundation

Here’s how we evaluated our options.

**OFF-THE-SHELF VS ROLL YOUR OWN**

1. Performance
  - Most off-the-shelf OOCSS frameworks are one-size-fits all, aka HEAVY.
  - More efficient / lightweight to roll your own framework, because you can limit to leanest, cleanest version possible.

2. Adaptability:
  - Don’t want to touch vendor code, so use override sheets for any customization
  - Vs. easily extensible if built in-house

3. Speed of development:
  - Wait for 3rd party maintainers to fix bugs, update things
  - Vs. Manage dependencies, updates internally

---

### CASE STUDY: Flatiron School’s Learn CSS Framework

**ROADMAP**

1. Take visual inventory of app’s UI/UX
2. Build component OOCSS library
3. Rewrite CSS and markup
4. Onboard your team

---

### STEP 1: VISUAL INVENTORY

Essentially pattern recognition. Our example: the Learn.co landing page.

Break down design into recognizable patterns. Start with base layout. Then look for containers. Then look for objects.

Once you’ve identified repeating patterns, translate them into CSS.

---

### STEP 2: COMPONENT LIBRARY

Modular components defined through strong naming conventions.

**MODULAR COMPONENTS:**

Compartmentalized. Individual classes should tend to do small things. In combination, classes can create complex looks, but complexity should come from the convergence of simple styles.

Portable. Our container and object elements should fit neatly into any column width. For the most part, one should be able to grab a contained HTML element from one template and plop it into another template with no stylistic issues.

**STRONG NAMING CONVENTIONS**

Shared semantic syntax =>  

- Less confusion  
- More consistency

Use abstract, generic class names.

Class names should very rarely reference their location or actual use-case, but should instead refer abstractly to the role they play in creating a layout. For instance `.homepage-project-list` is a poor class, but `.block-list` is a good name.

We chose BEM syntax to define classes that are verbose, accurate, and clear. BEM stands for `.block__element--modifier`.

Group related classes according to a root namespace. Single namespace is at the front of every class in the stylesheet makes it easy to find where a particular style is located, and to affect it locally without accidents.

Nesting is not necessary, because the BEM classes are already at the perfect level of specificity. In fact, nesting would bloat the final stylesheet, because each BEM class directly copies its parent class in its own name, duplicating the same string for each class in the chain. Nesting is useful for creating contextual or alternate styles, but should only be used when required.

Component library contains 4 scopes:

**LAYOUT COMPONENTS**

- HIGHEST LEVEL
- DEFINES GRID
- USED ON EVERY PAGE
- MOST RIGID STYLES bc least likely to change

Baseline styles for the grid, headers, footers, overlays, etc. Because they are one-off objects which must be rendered with great precision, Layout classes are often exceptions to the object-oriented paradigm and are created with efficient, powerful CSS. However, they do often create boxes in which our object-oriented elements may sit.

**CONTAINER COMPONENTS**

- Elements built to contain other elements, like lists or modules.
- In charge of SPACING, BACKGROUND, BORDERS, ALIGNMENT.

**OBJECT COMPONENTS**

- LOWEST LEVEL => most elemental component
- Small, atomic pieces at the very bottom of the DOM tree
- Rarely contains other objects; always inside a container

Most quarantined part of our CSS framework:

- no knowledge of or effect on its container
- rules don’t "infect" other things on the page

**GLOBAL COMPONENTS**

Use these sparingly to make precision adjustments.

---

### STEP 3: REWRITE

See code samples.

1. Start by outlining the basic container + objects
2. Semantic BEM syntax provides information on relationship between elements (parent vs. child, nesting level)
3. Place objects inside container

End product: encapsulated, reusable block of markup that's ready to drop into any view

---

But what about your JAVASCRIPT?

This project required us to rewrite all CSS and markup, which means we rewrote our JS selectors, too.

Javascript needs DOM element to hook into. Agreed on the following shared syntax: any JS selector element prefixed with `.js--`, for example `.js--toggle`.

- `.js--` classes have no styles attached to them
- Used for JS selectors only

---

### STEP 4: ONBOARDING

**PHASE 1**

- New stylesheets and markup shipped behind feature flag
- Training session on BEM syntax, OOCSS basics

**PHASE 2**

- QA testing: feature turned on for select user groups
- Training session on rewriting existing markup
- Identify & assign views for “hands-on learning” rewrites

**PHASE 3**

- Launch: feature 100% live
- Ship style guide with full framework documentation
- Pair with teammates as needed on new feature builds

---

### CHALLENGES

**SCOPE CREEP**

- Originally prototyped for 3 user-facing views
- Grew to cover ALL user-facing views
- Packaged with JS refactor

**NEOPHOBES**

- Change is scary
- Some people too comfortable doing things the “old way”
- Wary of CSS, so extra wary new system

Good news is we won everyone over once they started actually using the framework.

**OUTLIERS**

- New designs may introduce “one-off” elements
- Engineer's responsibility to push back on design/product before adding any new CSS
- Product/design need to justify introduction of stylistic inconsistencies

---

### RESULTS

**UX GAINS**

- Faster pageloads
- More consistent UI
- Greater parity with mocks

**DEVELOPMENT GAINS**

- Ship faster: BUILD NEW VIEWS WITHOUT WRITING ANY NEW CSS
- reuse markup with predictable results
- More accessible to those without extensive frontend experience
- Prototyping becomes more rapid and more accurate to the final product

**PERFORMANCE GAINS**

Payload

  - Native stylesheets
    - LEARN_V1: 213 sheets, 1.3 MB
    - LEARN_V2: 31 sheets, 193 KB

  - Vendor stylesheets
    - LEARN_V1: 116 sheets, 848 KB
    - LEARN_V2: 6 sheets, 49 KB

  - Total stylesheets
    - LEARN_V1: 329 sheets, 2.1 MB
    - LEARN_V2: 37 sheets, 242 KB


Page load benchmarks  
Average render times (source: Librato)

  - TRACK#SHOW
    - LEARN_V1: 262 ms
    - LEARN_V2: 108 ms

  - LESSON#SHOW
    - LEARN_V1: 98 ms
    - LEARN_V2: 48 ms

  - PUBLIC_LESSON#SHOW
    - LEARN_V1: 40 ms
    - LEARN_V2: 21 ms

---

### QUESTIONS?

Contact Kate Travers // [kate@flatironschool.com](mailto:kate@flatironschool.com) // [@kttravers](https://www.twitter.com/kttravers) // github: [ktravers](https://www.github.com/ktravers)

### ADDITIONAL RESOURCES

Nicole Sullivan’s OOCSS  
- https://github.com/stubbornella/oocss/wiki
- http://confreaks.tv/presenters/nicole-sullivan

BEM Syntax  
- https://css-tricks.com/bem-101/
- https://en.bem.info/method/key-concepts/
