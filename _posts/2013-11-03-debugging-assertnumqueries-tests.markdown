---
layout: main
title: "Debugging assertNumQueries tests"
description: "How to see exactly what queries have run in a Django test"
---

<span class='text-muted pull-right post-date'>{{ page.date | date:"%A %-d %B %Y" }}</span>
# Debugging assertNumQueries tests<br><small>How to see exactly what queries have run in a Django test</small>

Tests with `assertNumQueries` failing? Want a way to inspect your SQL queries in your tests? I have an answer.

I expect you have a test that looks something like this:

    class GoodTestCase(django.test.TestCase):
        def fantastic_test(self):
            with self.assertNumQueries(2):
                thing.do_fantastic_stuff()  # Queries happen here.

..and unexpectedly you're getting an error that looks a bit like this:

    FAIL: fantastic_test (greatproject.tests.GoodTestCase)
    ------------------------------------------------------
    Traceback (most recent call last):
      File "/greatproject/tests.py", line 130, in fantastic_test
        thing.do_fantastic_stuff()
      File "/path/to/django/test/testcases.py", line 105, in __exit__
        executed, self.num

    AssertionError: 3 queries executed, 2 expected

Okay, so we know something's wrong, but we don't know what. How do we find out what that extra query is? Well, Django can show you the queries run, so lets throw some debug code in the test to print them out...

    from django.conf import settings
    from django.db import connection

    class GoodTestCase(django.test.TestCase):
        def fantastic_test(self):
            with self.assertNumQueries(2):
                settings.DEBUG = True  # We can see the queries with DEBUG on.
                connection.queries = []  # Reset the list of recorded queries.
                thing.do_fantastic_stuff()  # Queries happen here.
                print(connection.queries)  # Print out our recorded queries!
                settings.DEBUG = False  # Turn DEBUG back off.
