---
layout: post
title: "Use 'The Notes of GHC'"
description: "A nice way to avoid cluttering code with repeated comments"
---

I like to write code comments in the style of [The Notes of GHC][notes-of-ghc].

The basic idea is that you write the large (or repeated) comment once in a sensible place,
and reference it in the myriad other places where you may want the extra context.

Here's an example of one:

```python
# Note [CharField length 191]
# ===========================
#
# MySQL VARCHAR fields with an index or constraint and the utf8mb4 encoding
# are limited to 191 characters in length.
# This is because MySQL assumes that each char may be 4 bytes long,
# and the maximum length of an InnoDB index is 767 bytes.
#
# 767 / 4 = 191.75
#
# See https://www.grouparoo.com/blog/varchar-191
```

Now when you write code, you can "link" to the note without massive comments breaking up your reading of the code.
This is especially useful if you might be tempted to repeat comments in lots of places.

For example, the code below "links" to the note we defined above next to each `CharField`.
The note doesn't even have to be in the same file, as you can search for the heading to find it anywhere.

```python
class MyModel(models.Model):
    # Note [CharField length 191]
    value = models.CharField(max_length=191, ...)


class MyOtherModel(models.Model):
    # Note [CharField length 191]
    name = models.CharField(max_length=191, ...)
```

Thanks to [jml][jml] for bringing this style to my attention. I hope it's useful!

[jml]: https://jml.io/
[notes-of-ghc]: https://www.stackbuilders.com/blog/the-notes-of-ghc/
