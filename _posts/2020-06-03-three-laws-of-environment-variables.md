---
layout: post
title: "Three Laws of Environment Variables"
description: "An idea for how to prioritise defaults."
---
My guidelines for default env var values are:

1. A default must not injure production or, through omission, allow production to come to harm.
2. A default must ease local development except where such a default would conflict with the First Law.
3. A default must reflect the live environment as long as such reflection does not conflict with the First or Second Laws.

This format is inspired by [Isaac Asimov's Three Laws of Robotics][three-laws-robotics].

Like Asimov's Laws, I expect there are some odd cases when they're pushed to their limits.


[three-laws-robotics]: https://en.wikipedia.org/wiki/Three_Laws_of_Robotics
