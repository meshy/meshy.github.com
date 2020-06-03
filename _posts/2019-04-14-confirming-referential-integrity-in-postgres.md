---
layout: post
title: "Confirming referential integrity"
description: "Ensuring PostgreSQL foreign keys reference records that exist"
---

Normally, I expect a database to maintain [referential integrity][integrity]
--- when something in Table A points to something in Table B, we can trust it
to exist there.

Imagine my surprise when I tried to move some table data to another database
with `db_restore`, and saw this:

    pg_restore: [archiver (db)] COPY failed for table "table_a":
    ERROR:  insert or update on table "table_a" violates foreign key
            constraint "table_a_related_b_fk_table_b_id"
    DETAIL: Key (related_b)=(1337) is not present in table "table_b".

It turns out someone deactivated checks when deleting some objects in this
dataset, and some records foreign keyed objects that didn't exist!

I had to find out which records had this issue, and I had a bit of a false
start when trying to find them. I cancelled this query after 14 hours:

    -- Slow query
    SELECT id, related_b
    FROM table_a
    WHERE related_b NOT IN (
      SELECT id FROM table_b
    );

...and replaced it with this one which took around 30 seconds.

    -- Faster equivalent
    SELECT id, related_b
    FROM table_a
    WHERE related_b is NOT NULL AND NOT EXISTS (
       SELECT id FROM table_b
       WHERE table_b.id = table_a.related_b
    );

The Explain Extended blog does a good job of explaining [why the second query is
faster][sql].

That found the records causing my immediate issue, but I was concerned that
there might be issues in other tables, so I went looking for a way to ensure
the rest of the database was intact.


INSERT FINISHED PRODUCT HERE


    WITH relationships AS (
        SELECT conrelid::regclass
            AS "FK_Table",
          CASE WHEN pg_get_constraintdef(c.oid) LIKE 'FOREIGN KEY %'
               THEN substring(
                    pg_get_constraintdef(c.oid),
                    14,
                    position(')' in pg_get_constraintdef(c.oid))-14) END
            AS "FK_Column",
          CASE WHEN pg_get_constraintdef(c.oid) LIKE 'FOREIGN KEY %'
               THEN substring(
                    pg_get_constraintdef(c.oid),
                    position(' REFERENCES ' in pg_get_constraintdef(c.oid))+12,
                    position('(' in substring(pg_get_constraintdef(c.oid), 14))-position(' REFERENCES ' in pg_get_constraintdef(c.oid))+1) END
            AS "PK_Table",
          CASE WHEN pg_get_constraintdef(c.oid) LIKE 'FOREIGN KEY %'
               THEN substring(
                    pg_get_constraintdef(c.oid),
                    position('(' in substring(pg_get_constraintdef(c.oid), 14))+14,
                    position(')' in substring(pg_get_constraintdef(c.oid), position('(' in substring(pg_get_constraintdef(c.oid), 14))+14))-1) END
            AS "PK_Column"
          FROM pg_constraint c
          JOIN pg_namespace n ON n.oid = c.connamespace
         WHERE contype IN ('f', 'p ')
           AND pg_get_constraintdef(c.oid) LIKE 'FOREIGN KEY %'
    )
    SELECT relationships.FK_Table,
           relationships.FK_
    FROM table_a
    WHERE NOT EXISTS (
       SELECT id FROM table_b
       WHERE table_b.id = table_a.related_b
    );

[integrity]: https://database.guide/what-is-referential-integrity/
[sql]: https://explainextended.com/2009/09/16/not-in-vs-not-exists-vs-left-join-is-null-postgresql/
[fks]: https://dba.stackexchange.com/a/200426/164955
