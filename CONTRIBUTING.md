# Contributing

This is a learning project. The whole point is that I write it myself, so I'm not
taking pull requests that contain implementation code. That's not a judgement on
anyone's code, it's just that a working implementation handed to me removes the
exercise.

Issues, though, are very welcome:

- a gradient that's wrong, an edge case I missed, a validation rule that doesn't
  actually hold
- a derivation in `docs/` that's incomplete or hand-wavy
- numerical trouble: overflow, underflow, a tolerance so loose it can't fail
- a benchmark of mine you think is measured wrong, ideally with your own numbers
- a paper or set of lecture notes that covers something here properly

The most useful form is "this is wrong and here's why", not a diff. Something
like "the input gradient in `Linear` contracts over the wrong index, so it only
happens to work when the layer is square" is exactly right. If you can give me an
input and the output you'd expect, even better.

What I'll close: PRs with implementation code, anything that adds numpy or a
similar dependency, and issues from people who clearly haven't read the code.
Typos and broken formulas in the docs are fine as PRs.

If you want to build the same thing yourself, fork it, delete `src/`, and work
through [TODO.md](TODO.md). You'll get a lot more out of that than out of reading
my answers. If you do, open an issue and tell me where you went a different way,
I'd genuinely like to know.
