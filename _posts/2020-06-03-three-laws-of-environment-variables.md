---
layout: post
title: "Three Laws of Environment Variables"
description: "An idea for how to prioritise defaults."
---
I'm not 100% sure if these rules are always applicable, but I've been thinking
about them for quite a while, and I'm increasingly happy with them. Without
further ado:

My guidelines for default env var values are:

1. A default must not injure production or, through omission, allow production to come to harm.
2. A default must ease local development except where such a default would conflict with the First Law.
3. A default must reflect the live environment as long as such reflection does not conflict with the First or Second Laws.

For the uninitiated, this format is inspired by Asimov's Three Laws of Robotics.
