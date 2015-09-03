---
layout: post
title: SQLAlchemy and Django
author: Rebecca
---
At [BetterWorks](http://betterworks.com), we use the [Django ORM](https://docs.djangoproject.com/en/1.8/topics/db) for data persistence and have found it to be very convenient for modeling and basic querying. It's very easy to use and does a lot of the work for you when it comes to dealing with multiple tables in the database. However, it falls short for some of the more complex queries.

Instead we use [SQLAlchemy](http://www.sqlalchemy.org/) for these advanced read-only use cases. We use [django-sabridge](http://django-sabridge.readthedocs.org/en/latest/) to instantiate SQLAlchemy tables and attach the `Bridge()` instance to the [local thread](https://docs.python.org/2/library/threading.html#threading.local). One hiccup we've hit while unit testing is that Django models are created and destroyed inside a test transaction, therefore, we had to create a subclass of [SQLAlchemy Query](http://docs.sqlalchemy.org/en/rel_1_0/orm/query.html#sqlalchemy.orm.query.Query) to execute queries inside the same database transaction.

Two example use cases of SQLAlchemy in our application are filtering by a function that requires joining a table with itself, and recursive [Common Table Expressions (CTE)](http://www.postgresql.org/docs/9.4/static/queries-with.html).

####Joining Tables

Let's say I have a Goal model that has a progress field and a related parent field also of type Goal,

```py
class Goal(models.Model):
    progress = models.IntegerField()
    parent = models.ForeignKey('self', related_name='children')
```

and we want to filter by this formula:

`2 * goal.progress <= goal.parent.progress`

Filtering by this function is complex in Django ORM because it requires a formula and a join, and aggregates don't handle this easily.  It is possible to use `queryset.extra()` to do this by using a `select_related` to get the second reference to the goal table.  The Django ORM query will automatically join the goal table, named `goal`, with itself and name the second table `T2`:

```py
Goal.objects.all()
    .select_related('parent')
    .extra(select={'compare_progress':
        'SELECT * FROM T2 WHERE 2 * goal.progress <= T2.progress'})
```

This is obviously a huge hack and strongly discouraged. We could do the above query with much more control in raw SQL:

```sql
SELECT * FROM goal
JOIN goal AS parents ON goal.parent_id = parents.id
WHERE 2 * goal.progress <= parents.progress
```

Dealing with raw strings isn't very friendly nor safe, so we rewrite this query using SQLAlchemy.

```py
goal_table = bridge[Goal]
parents_table = aliased(goal_table, name='parents_table')

session.query(goal_table)
    .join(parents_table, goal_table.c.parent_id == parents_table.c.id)
    .filter(2 * goal_table.c.progress <= parents_table.c.progress)
```

####Recursive CTEs

Another really useful example of the benefits of SQLAlchemy is recursive CTEs.  These are impossible with Django ORM and they can improve performance a lot in certain use cases. For example, if we want to get all of a goal's children in one query (including grandchildren, etc.).  The SQL would look like this:

```sql
WITH recursive children AS (
  -- start with the selected goal
  SELECT id, name
  FROM goal
  WHERE id = 1
  UNION
  -- unioned with children of all the goals in children
  SELECT goal.id, goal.name
  FROM goal, children
  WHERE goal.parent_id = children.id
)
```

Then we can query into the children table.  Unlike in Django ORM, we can duplicate this query in SQLAlchemy and this gives us way more options when writing queries:

```py
goal_table = bridge[Goal]
children = session
    .query(goal_table.c.id, goal_table.c.name)
    .filter(goal_table.c.id == 1)
    .cte(name='children', recursive=True)
children = children.union(
    session
        .query(goal_table.c.id, goal_table.c.name)
        .filter(goal_table.c.parent_id == children.c.id))
```

These are some cool examples of why SQLAlchemy has been a powerful tool for our application and how it has given us more control over the underlying SQL queries. SQLAlchemy has allowed us to build a complex goal filtering system, which helps our users find the information they need.
