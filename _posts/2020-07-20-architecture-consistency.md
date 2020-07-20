---
layout: post
title: Architecture's Biggest Benefit
comments: true
---

In my [last post](https://www.rstockbridge.dev/2020-06-06-architecture/), I went deep into common architecture choices for Android. Today, I want to share briefly what I consider to be the most important benefit of implementating an architecture style...

---

**Consistency** -> Pattern Recognition -> Ease of Understanding!

---

Of course we can discuss the merits and drawbacks of various architectural approaches (as I have done), and nowadays it's pretty clear that MVVM is the preferred choice in most circumstances. However, from my experience, it ultimately matters less **what** architecture you use and more **how consistently** you implement it.

When I first began working on an existing app that follows MVP architecture, I became confused by sometimes seeing the presenter telling the view what to do (standard behavior) and alternatively seeing the view telling the presenter what to do (non-standard behavior). Sometimes both behaviors were present in the same presenter/view pair! I found it difficult to follow the back-and-forth flow between presenter and view, and thought that maybe I just don't like MVP architecture.

With more experience, I now recognize that it was the lack of consistency that was confusing rather than the pattern itself. Implementing the architecture in the same way throughout the app saves time in the future by allowing the reviewer/next developer to use pattern recognition to follow what's going on. When the presenter is always the brain telling the view what to do, and the view is always just reporting naively back to the presenter, the behavior of the screen falls neatly into place. And that saves energy for more complicated considerations such as business logic, lifecycle, and so on.

So I strongly encourage you to follow and enforce a consistent implementation of the chosen architecture pattern. Less experienced and future developers on the project will thank you!