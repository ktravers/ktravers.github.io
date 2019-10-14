---
layout: post
title: It's `document.write(new Date().getFullYear());` - Update Your Footer
tags: [javascript]
---

As I was updating some content on my [personal website](http://www.kate-travers.com) today, I noticed the copyright date in the footer still read "2014". Yeah, it's May. Go me.

Sadly, I believe future-me from 2016 would probably make this same mistake, so to help myself out, I did a quick google search for "How to auto update year website", which yielded this extremely helpful site: [UpdateYourFooter.com](http://updateyourfooter.com/). It offers Javascript and PHP solutions, with JS being the easier option for my needs.

Just add the following wherever you want the year displayed:

```javascript
<script>
  document.write(new Date().getFullYear());
</script>
```

You can find more info on the Javascript Date object [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date). And best of all, this solution is [supported in all browsers except IE8](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date#Browser_compatibility). Finally, there's (of course) a longer discussion about this on [StackOverflow](http://stackoverflow.com/questions/4562587/shortest-way-to-print-current-year-in-javascript), for anyone who wants some additional reading.
