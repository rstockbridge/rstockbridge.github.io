---
layout: post
title: What I Wish I'd Known About Software Development As A Mathematician
comments: true
---

I've been a professional software developer for about two years, but I've been writing code in some form for nearly twenty years. I coded up mathematical algorithms as part of undergraduate numerical analysis projects, graduate research in stochastic optimization, and professional work as a data scientist. 

And what did these efforts have in common besides the nature of the work? All throughout I knew absolutely nothing about good software development practices.

For mathematicians (and many scientific researchers in general), writing code is simply a means to the end of generating publishable research, rather than a skilled craft to improve at. So I learned a lot about developing research ideas and persuading others of their value via papers, but was left to my own devices when it came to coding. I'm sure you can imagine the results.

This post will cover the best practices I wish I had been aware of when writing mathematical code. They're absolutely standard for developers, but given that many (most?) researchers have no training in software development I hope my perspective can be useful. Keep in mind it's been a while since my journey began, so expectations of researchers may have already improved significantly since I started grad school.

---

**Descriptive Variable Names**

This may seem the most obvious practice to a developer - but not necessarily to a mathematician, since mathematical notation commonly uses single letters such as `x`, `n`, and `p`. You might want to use those letters as variables in the code to match the written mathematical calculations. However, descriptive names can still be used elsewhere throughout the code for improved clarity, such as for intermediate and final quantities.

For example, the following pseudocode illustrates calculating the volume of a cylinder with radius `r` and height `h`:

```
areaOfBase = PI * r^2
volume = areaOfBase * h
return volume
```

Personally, I would write out `radius` and `height` rather than using `r` and `h` in this contrived example, but in more complex situations it may be clearer to match variable names with mathematical notation. As shown, it can also be beneficial to name the returned value for clarity (if not brevity) instead of just using `return areaOfBase * h`.

---

**Version Control**

When I started my PhD research, I was given a folder of C++ files from another student. He had inherited the code from our advisor, who had been given it by another professor, and so on and so on. (These files had been auto-converted from Fortran to C++ some years prior but that's another story.) I similarly passed the code on to other students. We each had our own copies for our particular work, with no version containing everything. It was also very difficult to learn about any prior changes to the code before we received it.

I didn't learn about version control in time to salvage that situation, but excitedly jumped at the chance to introduce Git to my project when working in industry. Even if you aren't sharing code with anyone else, it's so helpful to have a record of your work and be able to jump back and forth in time, try out new ideas on fresh branches, and use good old `git reset --hard` when it all goes wrong. Set up version control right at the start and don't look back.

---

**Tooling**

I was like a kid in a candy store the first time I heard about "jump to definition" and "jump to completion". Until I started using Android Studio about three years ago, I had done all my coding in a plain text editor. That means no linters, no auto-formatters, no ability to navigate the code, no mass renaming, and certainly no possibility of any more complicated refactoring. 

IDEs (Integrated Development Environments) have come a long way in recent years, and so I may not have been able to use all these tools back then anyway. But I didn't even know to ask the question. I just thought that the act of writing code was difficult by design, and it never occurred to me it could be easier or more efficient.

Search for the most common IDE for your particular language or application, and take advantage of all the conveniences it provides you.

---

**Writing Tests**

As a mathematician writing code, my main challenge was to correctly implement very complicated algorithms and calculations. The only goal of this code was to produce a correct output for a given input. I didn't need to worry about lifecyle, asynchronicity, configuration changes, or the other concerns that keep Android developers up at night. This would have been the perfect time to write tests for my code, whether unit tests for specific functions or integration tests for entire calculations. 

Instead, I remember adding print statements all over my code to verify my calculations (I didn't know anything about debugging, either). Statements I of course had to delete when I was satisfied (and possibly re-add if I needed to check again). 

As an Android developer, I take a pragmatic approach to testing, prioritizing business logic and scary calculations rather than testing everything. But I sure wish I'd had tests to build confidence in my research results.

---

Given that these four practices are taken for granted in professional software development, it may be surprising that they are not always mainstream in other fields. You might think that non-developers who write code may be less thoughtful or less capable. In my opinion, it comes down simply to a lack of awareness and different incentives. 

Scientific researchers and software developers live in mostly separate worlds, but there is a natural synergy - a computer is one of the best tools we have to solve hard problems. If you're a developer reading this post, my takeaway for you is that there is so much opportunity for professional developers to share what we've learned with others. And if you're a researcher, think about small changes you can make to your coding processes that will significantly improve productivity and the longevity of your code.

Finally, a massive shoutout to my husband, [Stuart Kent](https://www.twitter.com/skentphd), for opening my eyes to practices that make writing code fun again!
