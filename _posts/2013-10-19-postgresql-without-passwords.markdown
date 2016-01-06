---
layout: main
title: "PostgreSQL without passwords"
description: "How to set up PostgreSQL 9.1 for local user access/tests on Ubuntu Linux"
---

<header class='post-title'>
    <span class='date'>
        {{ page.date | date:"%A %-d %B %Y" }}
    </span>
    <h1 class='title'>PostgreSQL without passwords</h1>
    <p class='subtitle'>
        How to set up PostgreSQL 9.1 for local user access/tests on Ubuntu Linux
    </p>
</header>

Wouldn't it be nice if you didn't need to type out that nasty
`sudo -u postgres pqsl` *and* then enter a password every time you wanted to
use a local development database? Well, if you don't mind every Tom, Dick and
Harry having a peek at your data, you can get all that "security" balderdash
out from under your feet by granting superuser access to your normal login
user. `:D`

1. Add the following to your `/etc/postgresql/9.1/main/pg_hba.conf`,
**replacing `meshy` with your login name**. (If you don't know your username
you can get it by running `whoami` in your terminal.)

        host    all     meshy       127.0.0.1/32            trust
        local   all     meshy                               trust
        local   all     all                                 ident

2. Restart postgres

        sudo -u postgres service postgresql restart

3. Set up postgres.

    Run postgres:

        sudo -u postgres psql

    Create superuser and database.

        create user meshy;
        alter role meshy with superuser;
        alter role meshy login;
        create database meshy;

4. This step is not required, and should **NEVER be used in production**.
    If you are fond of the data in *any* of your databases using this install
    of postgres **Do Not Use This**.

    That said, if the data doesn't matter, and your only concern is speed, this
    should sharply increase database creation time and speed up your tests.

    Final warning: **[Do NOT use on machines with important data!](http://managing-geeks.blogspot.co.uk/2009/07/what-happens-when-you-turn-fsync-off-on.html)**
    You have been warned.

    Edit `/etc/postgresql/9.1/main/postgresql.conf` with:

        # WARNING! This is dangerous!
        # These settings are likely to cause corruption across all databases.
        fsync = off
        synchronous_commit = off
        full_page_writes = off

5. Profit.

        psql

I'm using this to run Django tests on Ubuntu 12.04 and 13.04, but I'm sure this is fit for many other purposes. Have fun, and happy hacking!
