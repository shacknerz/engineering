---
layout: post
title: Ditching Django REST Framework Serializers for Serpy
author: Clark
---

This is the story of how we ditched [Django REST Framework
Serializers](http://www.django-rest-framework.org/api-guide/serializers/) for
[serpy](https://github.com/clarkduvall/serpy) and got a performance boost in almost all of our API
endpoints.

Here at [BetterWorks](https://betterworks.com), we use a lot of tools to keep tabs on our app and
make sure our API is performing quickly and efficiently. In production, we use [New
Relic](http://newrelic.com), and locally we use tools like [Django Debug
Toolbar](https://django-debug-toolbar.readthedocs.org) and Python's
[profilers](https://docs.python.org/2/library/profile.html). Many times, an endpoint is executing
too many queries, or the queries are inefficient. These problems are usually pretty easy to solve.
Other times, our Python code is the problem.

As I was profiling some of our slower APIs, I went through my checklist:

  - Too many SQL queries happening? __NOPE__
  - Slow SQL queries happening? __NOPE__
  - A lot of obvious work being done in Python? __NOPE__

A typical view I was seeing this pattern for looked something like this:

```py
class GoalsView(BaseView):

    def get(self, user_id, *args, **kwargs):
        goals = Goal.objects.filter(owner_id=user_id)
        return Response(GoalSerializer(goals, many=True).data)
```

Simple right? So I got out one of my favorite tools
[RunSnakeRun](http://www.vrplumber.com/programming/runsnakerun/) to visualize the output from
cProfile. It turns out most of the work was being done serializing Django models into the format
needed for the response. We were using [Django REST Framework (DRF)
Serializers](http://www.django-rest-framework.org/api-guide/serializers/) for this step, and they
were the bottleneck for these APIs.

I took a look into the DRF serializer code, and saw that a lot of work was being done each time the
serializer was used. This was necessary because of the large amount of features the DRF serializers
support, but we didn't use most of these features.

After looking at this, I decided to write my own serialization library that was focused on
simplicity and performance. A couple days later I had the first version of
[serpy](https://github.com/clarkduvall/serpy). One of the key ideas in serpy was pushing as much
work as possible to the serializer's
[metaclass](https://docs.python.org/2/reference/datamodel.html#customizing-class-creation). This
means that when actual serialization happens, it is just a simple loop through the fields to be
serialized.

Using serpy, we had a big performance boost in endpoints where the bottleneck was serialization.
This graph shows a simple benchmark between serpy, DRF serializers, and Marshmallow (another popular
serialization framework):

![serpy benchmark](/public/img/2015-09-04-ditching-django-rest-framework-serializers-for-serpy/benchmark.png)

More benchmarks can be found [here](http://serpy.readthedocs.org/en/latest/performance.html).

We have been using serpy in production for a few months now, and haven't had any more bottlenecks on
serialization. Django REST Framework is still used for other parts of our app, but serialization
required a more performant library.
