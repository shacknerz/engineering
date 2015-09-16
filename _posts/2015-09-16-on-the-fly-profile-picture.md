---
layout: post
title: On The Fly Profile Picture
author: di
---

In our app, users are assigned a default profile picture upon account creation. We used to have a
set of cartoonish creature pictures that are randomly selected for each user based on their user id.

We used to assign a random picture from the following set via `user_id % 15`:
![old pictures](/public/img/2015-09-16-on-the-fly-profile-picture/old.png)

*Trivia: the first two images were secret ones that only devs can obtain*

As you can see, we don't have a big selection of pictures so with bigger customer organizations,
it can be hard to distinguish users. To solve this, our design team came up with a more personalized
approach to default pictures using people's initials and a random color from our palette. This also
means we would have to **render the pictures on the fly using dynamic information** about the user.

![new pictures](/public/img/2015-09-16-on-the-fly-profile-picture/new.png)
*a sample of some images from our team*

The business logic in this project is very simple, so we decided on a [NodeJS](https://nodejs.org)
server using [express](http://expressjs.com/) and image rendering using
[node-canvas](https://github.com/Automattic/node-canvas). The [actual rendering code](https://github.com/BetterWorks/defaulter/blob/master/routes/index.js)
is less than 50 lines and we had deployed on [Heroku](https://heroku.com) within hours.

*Trivia: to render Chinese Japanese Korean characters properly, we detect `char > '\u2E7F'`
and switch to a font that supported CJK characters*


We learned a lot about server side canvas drawing while building this and our production service
has been averaging a blazingly fast 26ms response time.
![performance](/public/img/2015-09-16-on-the-fly-profile-picture/perf.png)

Feel free to check out the open sourced code at
[github.com/BetterWorks/defaulter](https://github.com/BetterWorks/defaulter).

We even included a convenient Heroku deploy button so that you can get up and running with your own
service in seconds.
