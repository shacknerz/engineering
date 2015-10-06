---
layout: post
title: Five Lessons Learned the Hard Way
author: di
published: false
---

![anniversary](/public/img/2015-10-06-five-lessons-learned-the-hard-way/anniversary.jpg)

> [A startup is a company designed to grow fast.](http://www.paulgraham.com/growth.html) - Paul Graham

It's been two years since we started [BetterWorks](http://betterworks.com). As we scale
the company and strive to accomplish more ambitious goals, we've often found that the things holding
us back are not raw technical execution but the more non-technical aspects of building products as a team.

Here are a few lessons/heuristics we've learned first handedly on product building. Think of these
as guiding principles rather than absolute rules to live by for an engineering + product team of roughly
~25.

### Keep project teams small
![communication](/public/img/2015-10-06-five-lessons-learned-the-hard-way/communication.png)
Imagine each dot as an engineer or product manager or designer who is responsible for a portion of a
project. From a geometric perspective, it's obvious that the amount of work necessary to keep just
that extra person in sync with the rest of the group becomes very expensive, very quickly.

True fact, we've had one project with 4 engineers and inefficiencies that came up were wasted work,
being blocked, and changes not communicated to everyone.

*Unless we've committed to a hard external deadline, we'd rather roll out the project in smaller chunks
over a longer period of time with less people involved.*

### Think in terms of opportunity cost
![opportunity cost](https://imgs.xkcd.com/comics/working.png)
For the first year of the company, the entire engineering team could fit into a single SUV. Being
such a small team forced us to ruthlessly prioritize features, triage bugs, and address overruns.
For every thing that we intended to do, it was a conscious decision not to tackle the next thing.

Over time, as our team grew and our product matured, we've lost some of that ruthlessness when it comes
to tradeoffs. True fact, recently we spent 3 story points and incurred technical complexity on a feature
that less than 1% of our users uses.

*Everyone must be aware that the act of choosing X is the equivalent of rejecting Y.
Have a list of known impactful improvements that can be done and use that as the litmus test.*

...
