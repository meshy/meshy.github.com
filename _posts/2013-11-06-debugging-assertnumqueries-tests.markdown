---
layout: main
title: "Debugging assertNumQueries tests"
description: "How to see exactly what queries have run in a Django test"
---

<span class='text-muted pull-right post-date'>{{ page.date | date:"%A %-d %B %Y" }}</span>
# Debugging assertNumQueries tests<br><small>How to see exactly what queries have run in a Django test</small>

<div class="alert alert-info">
    <strong>Update:</strong>
    Thanks to <a class='alert-link' href='https://twitter.com/dominicrodger/status/412713091877437440'>Dominic Rodger</a> this behaviour is now in Django 1.7+.
</div>

Tests with `assertNumQueries` failing? Want a way to inspect your SQL queries in your tests? I have an answer.

I expect you have a test that looks something like this:

    class GoodTestCase(django.test.TestCase):
        def test_fantastic_stuff(self):
            with self.assertNumQueries(1):
                thing.do_fantastic_stuff()  # Queries happen here.

..and unexpectedly you're getting an error that looks a bit like this:

    FAIL: test_fantastic_stuff (greatproject.tests.GoodTestCase)
    ------------------------------------------------------
    Traceback (most recent call last):
      File "/greatproject/tests.py", line 130, in test_fantastic_stuff
        thing.do_fantastic_stuff()
      File "/path/to/django/test/testcases.py", line 105, in __exit__
        executed, self.num

    AssertionError: 2 queries executed, 1 expected

Okay, so we know something's wrong, but we don't know what. How do we find out what that extra query is? Well, Django can show you the queries run if you just ask it nicely. Lets throw some debug code in the test to print them out...

    from django.db import connection

    class GoodTestCase(django.test.TestCase):
        def test_fantastic_stuff(self):
            with self.assertNumQueries(1):
                # Get num queries run before.
                so_far = len(connection.queries)
                thing.do_fantastic_stuff()  # Queries happen here.
                for query in connection.queries[so_far:]:
                    print(query['sql'])  # Print our queries

When you run this, you should get the executed queries printed in your test output:

    SELECT (1) AS "a" FROM "great_model" WHERE "great_model"."id" = 15  LIMIT 1
    UPDATE "great_model" SET "field" = 1 WHERE "great_model"."id" = 15

Nice, eh? That should be enough to set you on the path of debugging the issue, but I feel like going a little further (too far?) and making it pretty. Wouldn't it have been nicer to run the following? (Note the extra context manager.)

    class GoodTestCase(django.test.TestCase):
        def test_fantastic_stuff(self):
            with self.assertNumQueries(1), PrintQueries():
                thing.do_fantastic_stuff()  # Queries happen here.

I think so too, so here's my definition of `PrintQueries`:

    from django.db import connection

    class PrintQueries(object):
        def __enter__(self):
            # Get num queries run before.
            self.so_far = len(connection.queries)

        def __exit__(self, type, value, traceback):
            for query in connection.queries[self.so_far:]:
                print(query['sql'])  # Print our queries.

I hope you find it useful! `:)`
