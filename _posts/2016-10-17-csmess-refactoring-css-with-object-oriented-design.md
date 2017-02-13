---
layout: post
title: CSMess to OOCSS - Refactoring CSS With Object Oriented Design
---

[Presented at Fullstack Toronto Conf. 2016](https://eventmobi.com/fstoco2016/agenda/141857/806561)

## Slide deck

<iframe src="https://docs.google.com/presentation/d/12H2MLnGdaFN2xk-MRVyIRX4j77zQeAHytcUmFcQ6b2Q/embed?start=false&loop=false&delayms=5000" frameborder="0" width="560" height="344" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

## How Object-Oriented Design Saved Our CSS (and Site Performance)

Back in January 2016, our CSS was easily the messiest part of the [Learn.co](https://learn.co) codebase… something that probably sounds familiar to many other web developers out there. Last spring, we budgeted time and resources to overhaul it. I led that project. Here's what we learned.

Before diving in, let’s clarify what we’re talking about when we talk about object-oriented design. Object-oriented code is:

- **Modular:** reusable
- **Encapsulated:** self-contained with minimal dependencies
- **Maintainable:** easy to work with, which is especially important for smaller dev teams like ours

[Learn.co](https://learn.co) is built on a Rails backend, and Rails strongly encourages writing code in an object-oriented paradigm. But how does object orientation apply to stylesheets? As good Rubyists, we tried to follow object orientation with our CSS, but we were doing it wrong. To illustrate, let’s take a look at our old stylesheets directory tree.

![Stylesheet Directory Tree]({{ site.baseurl }}/assets/css-tree.png "Stylesheet Directory Tree")

Very similar to the Rails boilerplate setup, we had one stylesheet per view, with styles namespaced under a top level ID. This approach is fine when you’re starting out, but over time, it can get out of hand. Just look at where ours was pre-rewrite:

![Stylesheet Directory Bundle]({{ site.baseurl }}/assets/css-bundle.png "Stylesheet Directory Bundle")

As you might be able to guess from this sizeable bundle, this approach is NOT object oriented.

**It’s not modular.** Styles are page-specific, so opportunities for code sharing are missed, and you end up with lots of code duplication. Plus, if you redefine styles on a page-by-page basis, you inevitably end up with inconsistencies in implementation and the visual interface itself.

**It’s not encapsulated.** Nesting everything under a top-level ID kicks things off with a high level of specificity that you’ll be battling throughout the rest of your stylesheets. This setup also makes it hard to predict what’ll happen when you change a class name or reorganize the sheet.

**It’s not maintainable.** Maintaining this kind of CSS is PAINFUL. Writing new CSS for every new view slows down development. It slows down page performance, too, contributing to a massive assets payload. Lots of new CSS means lots of churn, again increasing the risk of inconsistencies in code and design. And good luck trying to onboard into that mess.

We knew our CSS was broken. But what’s the solution?

Enter OOCSS.

![OOCSS]({{ site.baseurl }}/assets/briefcase.gif "OOCSS")

After consulting with our good pals at [Cantilever.co](http://cantilever.co/), we committed to an OOCSS solution. But now we had another decision to make: use an existing OOCSS framework or roll our own?

There are lots of off-the-shelf frameworks out there, including [Twitter Bootstrap](https://getbootstrap.com/2.3.2/), [Yahoo Pure](https://purecss.io/), and [Zurb Foundation](http://foundation.zurb.com/) (some of the most popular options). We considered three main factors.

**Performance.** Most off-the-shelf OOCSS frameworks are one-size-fits-all, aka HEAVY. We thought it’d be more efficient to roll our own framework, because we could limit it to the leanest, cleanest version possible.

**Adaptability.** Pre-rewrite, we relied on a good amount of vendor styles. Any time we needed to customize something, we’d create an “override” sheet, adding more payload and more specificity conflicts. So we loved the idea of sticking to easily-adaptable, in-house styles only.

**Speed of development.** With third party libraries, you have to wait for the maintainers to make updates and fix bugs, and their schedules don’t always align with yours. We ship fast, so being able to manage dependencies and handle updates internally was super appealing.

![OOCSS Options]({{ site.baseurl }}/assets/oocss-solution-table.png "OOCSS Options")

We had a big task in front of us: writing our own Learn.co-customized OOCSS framework. We broke the process out into a four step roadmap:

1. Take visual inventory of app’s UI/UX
2. Build component OOCSS library
3. Rewrite CSS and markup
4. Onboard the rest of the team


### STEP 1: VISUAL INVENTORY

The first thing you’ll want to do is take a visual inventory of your entire site. The goal is to break down the design into recognizable patterns. We can use the [Learn.co](https://learn.co) landing page as an example.

![Learn.co homepage]({{ site.baseurl }}/assets/learn-co-homepage.png "Learn.co homepage")

Start with the base layout - your header, footer, sidebar, etc.

![Learn.co layout scope]({{ site.baseurl }}/assets/learn-layout-scope.png "Learn.co layout scope")

Then take one step inward and look for “containers”. Look inside those containers for recurring “objects”.

![Learn.co container scope]({{ site.baseurl }}/assets/learn-container-scope.png "Learn.co container scope")


### STEP 2: COMPONENT LIBRARY

After you’ve identified repeating patterns, the next step is to translate them into CSS. Working with [Cantilever.co](http://cantilever.co/), we designed a system that groups stylesheets into four main scopes:

**Layout components** set the baseline styles for the page grid, headers, footers, overlays, etc. Since these classes require the most precision and are the least likely to change, they tend to contain more rigid, powerful rules than the rest of your components.

![Layout Components]({{ site.baseurl }}/assets/layout-components.png "Layout Components")

**Container components** are the real workhorse of your OOCSS system. They’re built to contain other elements, like lists or images, and they’re in charge of handling spacing, background, borders, and alignment.

![Container Components]({{ site.baseurl }}/assets/container-components.png "Container Components")

**Object components** handle styles for the smallest, discrete elements at the very bottom of the DOM tree, like buttons, links and images. They’re always the containee, not the container, and they should have the same, predictable behavior no matter what container they’re placed inside.

![Object Components]({{ site.baseurl }}/assets/object-components.png "Object Components")

**Global components** are your cleanup classes, little helpers that allow you to make precision adjustments. Examples include display utilities that hide/show elements at certain breakpoints. These should be applied sparingly to prevent inconsistencies in implementation and/or UX.

![Global Components]({{ site.baseurl }}/assets/global-components.png "Global Components")

Now our stylesheets directory looks something like this:

```
└── assets
      └── stylesheets
          ├── layout
          │   ├── site-header.scss
          │   ├── site-footer.scss
          │   └── site-main.scss
          ├── containers
          │   ├── list.scss
          │   ├── media-block.scss
          │   └── module.scss
          ├── objects
          │   ├── button.scss
          │   ├── image-frame.scss
          │   └── input.scss
          ├── global
          │   ├── special.scss
          │   ├── typography.scss
          │   └── vars.scss
          └── application.scss
```

When it comes to actually writing the CSS, you’ll want to focus on writing modular components defined through strong naming conventions.

**Modular components** are compartmentalized. Individual classes should do small things. Combine these small classes to get the big, complex looks you want. They should also be portable; our container and object elements should fit neatly into any column width. Your end goal should be to have a system where you can grab a block of markup from one template and drop it into another with complete confidence in what the end result will look like.

**Strong naming conventions** provide structure. CSS gives you so much freedom when it comes to naming things, so it’s important to develop a shared semantic syntax for your class names. Your devs should be able to glance at a class name and have a reasonable idea of what the element looks like on the page. You’ll want to use abstract, generic class names. Don’t base names on a specific location or use-case. Instead, class names should reference their visual role. For example, `.homepage-project-list` is not so great, but `.bubble-list` is pretty good.

We chose [BEM syntax](https://en.bem.info/method/key-concepts/) for our system, which gives us self-documenting class names that are accurate and clear, if a little verbose.

BEM stands for "Block-Element-Modifier":

```scss
/* Block component */
.list

/* Child element of parent block */
.list__card

/* Modifier that changes the style of the block */
.list--accordion
.list--spacing-large

```

When you use BEM naming conventions for CSS, think of your class names like objects. Apply the root namespace to the parent element - in the above example, an unordered list `<ul>`. Child elements are designated by the double underscore, and any modifiers are indicated with a double dash. Note that we’re not nesting any of these classes; there’s no need, since we can read the relationship between objects from the name itself. Plus this way, we avoid any specificity battles because everything’s on the same level.


### STEP 3: REWRITE

We’ve designed our system. Now it’s time to rewrite all of our markup. We’ll look at one of the most commonly-used patterns as an example: `.media-block`.

![Media Block]({{ site.baseurl }}/assets/media-block.png "Media Block")

`.media-block` has three main parts: the parent `.media-block` class and two child classes: `.media-block__content` and `.media-block__media`.

![Media Block with markup]({{ site.baseurl }}/assets/media-block-with-markup.png "Media Block with markup")

When writing the markup, start by outlining the basic container and objects. Rely on the semantic BEM syntax to provide information the the relationships between elements - parent <> child, nesting level, etc.

```html
<div class="media-block">

  <div class="media-block__media">

    <!-- circle with user’s profile picture -->

  </div>

  <div class="media-block__content">

    <!-- text object: user name -->
    <!-- text object: time ago -->
    <!-- text object: action completed by user -->

  </div>

</div>
```

After you’ve built the frame, add text and media elements to fill out the card. Your end product is a fully encapsulated, reusable block of markup that’s ready to drop into any view.

```html
<div class="media-block">

  <div class="media-block__media">
    <div class="image-frame image-frame--border-radius-full">
      <img class="image-frame__image" src="img.png"/>
    </div>
  </div>

  <div class="media-block__content">
    <span class="heading heading--level-5">Brent...</span>
    <span class="heading heading--level-6">5 min ago</span>
    <h5 class="heading heading--level-5">Built...</h5>
  </div>

</div>
```

![Record Scratch]({{ site.baseurl }}/assets/record-scratch.gif "Record Scratch")

::record scratch::

But what about our JAVASCRIPT?

This project required us to rewrite all of our CSS and markup, which means we had to rewrite our Javascript selectors, too. We relied on semantic BEM syntax here as well, adopting the following syntax for any Javascript selector element: descriptor prefixed with `.js--`. For example, `.js--toggle`.

This was also a good opportunity to decouple our JS and CSS. Moving forward, `.js--` classes have no styles attached to them. They’re reserved for JS selectors only.


### STEP 4: ONBOARD THE TEAM

We onboarded our team in three phases.

In phase 1, we shipped new stylesheets and markup as dark code, “hidden” behind a feature flag. I also held a brief training session with the team on BEM syntax and OOCSS basics.

In phase 2, we QA’d the feature while gradually turning the feature flag on for a greater and greater percentage of our users. I held a second training session with the team where we did some hands-on practice rewriting existing markup.

In phase 3, we flipped the feature live for 100% of our users. I shipped a style guide with full framework documentation, and then paired with teammates as needed on new feature builds, until the whole team was comfortable with the new system.

In all, the process took about two and half months, with three dedicated devs at peak, eventually scaling back down to one.

### CHALLENGES

This project wasn’t without its challenges.

The primary beast we faced was **scope creep**. Originally, this project was prototyped for three user-facing views. It grew to cover ALL user-facing views. And since we had to rewrite our JS selectors anyway, we ended up refactoring most of our Javascript alongside our markup and CSS. Letting things expand like this easily added another few weeks to the project.

After scope creep, the second biggest challenge we faced was something I'll call **neophobia - fear of change**. Change is scary, and some people on our team were very comfortable doing things the old way (maybe too comfortable). Some folks were also pretty wary of CSS in general, and those deep-seeded biases were hard to overcome. But I’m happy to say once everyone started actually using the system, they were completely won over.

Finally, we had to and continue to be vigilant against **“outliers”**. Whenever designs introduce one-off elements, or something otherwise inconsistent with our existing UI, it’s our responsibility as engineers to push back before adding any new CSS. Product/design now needs to justify the introduction of stylistic inconsistencies.

### RESULTS

So what did all this work earn us?

First, we saw **impressive UX gains.** Paring down our code gave us faster pageloads, a more consistent UI, and greater parity with our product mocks.

We also enjoyed some **nice developer gains.** This plug-and-play system allows our team to ship faster. Now we can build new views without writing any new CSS. We can reuse markup with predictable results, so prototyping becomes faster and better reflects the final product, and you no longer need much front-end experience to be able to put up a pretty view quickly.

Maybe most impressive our all were our **performance gains.** Below are unzipped, unminified size comparisons for our assets payload. You’ll see we were able to reduce our payload 10x.

![Results - File Count]({{ site.baseurl }}/assets/results-file-count.png "Results - File Count")

Here’s average page render times for our most heavily trafficked views. We cut most of these in half.

![Results - Page Load]({{ site.baseurl }}/assets/results-page-load.png "Results - Page Load")

We devoted a lot of resources to this project, but as you can see from these big wins, it was well worth it. The biggest lesson learned was not to wait. You can save yourself a lot of pain by applying OO design to your CSS from the start.


### ADDITIONAL RESOURCES:

Nicole Sullivan’s OOCSS  
- https://github.com/stubbornella/oocss/wiki
- http://confreaks.tv/presenters/nicole-sullivan

SMACSS Style Guide  
- https://smacss.com/

BEM Syntax  
- https://css-tricks.com/bem-101/
- https://en.bem.info/method/key-concepts/
