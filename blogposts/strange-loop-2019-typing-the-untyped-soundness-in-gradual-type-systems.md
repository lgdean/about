---
title: "Strange Loop 2019 - Typing the Untyped: Soundness in Gradual Type Systems"
description: "Recent years have seen an explosion of gradual type systems and superset languages that add types to previously untyped languages: TypeScript & Flow for Javascript, MyPy and Pyre for Python, Hack and PHP7 for PHP, Sorbet for Ruby, and many more. Implementing these type systems involves making tradeoffs between soundness (catching as many errors as possible) and completeness (not rejecting valid programs) that fundamentally impact the usability and usefulness of the type system."
author: Logan Dean
authorUrl: https://twitter.com/lgdean/
publishDate: 2019-09-13T00:00-10:20
tags: [
  strange-loop
]
slug: strange-loop-2019-typing-the-untyped-soundness-in-gradual-type-systems
heroImage: /blog/strange-loop-thumbnail-square-v2.jpg
published: false
---

<div class="container p-0 liveblog-presenters">
  <div class="row m-0">
      <p class=" mr-12 m-0">
        <span class="liveblog-presenters__name">Ben Weissmann</span>
        <a href="https://github.com/benweissmann" target="_blank" title="GitHub"><i class="fa fa-github pr-2"></i></a>
        <a href="http://benweissmann.com" target="_blank" title="Speaker's site"><i class="fa fa-globe pr-2"></i></a>
      </p>
  </div>
</div>

---

## Overview

Recent years have seen an explosion of gradual type systems and superset languages that add types to previously untyped languages: TypeScript & Flow for Javascript, MyPy and Pyre for Python, Hack and PHP7 for PHP, Sorbet for Ruby, and many more. Implementing these type systems involves making tradeoffs between soundness (catching as many errors as possible) and completeness (not rejecting valid programs) that fundamentally impact the usability and usefulness of the type system.

---


Ben "Fuzzy" Weissmann works at a company called Tulip.


He has noticed, there and elsewhere, that the same problems happen over and over again:
sometimes the type system doesn't catch a bug they might have hoped it would;
other times, it doesn't allow code that should be fine.

And when they go in to read about it, the documentation is complicated and confusing.

Two JS type systems:
TypeScript and flow have different goals.
TypeScript TODO finish sentence

MyPy for Python, Sorbet for Ruby.

Designed to be gradual: you can add onto an existing codebase.

To contrast, look at statically-typed languages from Day 1: Go, Rust, Java.

Slides at https://bit.ly/sl-types

# Soundness and Completeness

First, let's look at some terminology.

Soundness:

Completeness:

The first two, we want from *any* type system;
the last one is specific to grdual type systems:
to work well with existing language features and idioms
that were not designed with type-checking in mind.

## Two simple examples in Javascript

### Unsoundness

### Incompleteness

This code will always be OK, but the type system doesn't understand that.

## Three Case studies in JS

### Array Indexing

In JS, if you read outside the bounds of an array, you get `undefined`.

So if we really wanted TypeScript to check us meticulously,
we coiuld tell it, it can be `number` or `undefined`.
So why doesn't TS always do this?
Well, the biggest problem is things like for loops:
it is always safe to do this, and it would be really annoying
if TS told us it could be undefined!
So it is a trade-off.

If you want more soundness, undef;
if more completeness, trust programmer.

#### Sorbet

Why does Sorbet do this?
In Ruby, we dont' generally read out of arrays:
we use methods like `each` to iterate over collections.

With this version of Nilable, it works OK.

#### MyPy

MyPy doesn't have this issue at all:
in Python, it's an immediate runtime error to index OOB.

#### Recap

Would be too incomplete, too frustrating in JS.
But in Ruby, can make a different trade-off, due to idioms.
Other langs don't care: immediate error (at runtime).

## Refinement and Invalidation

Type system will understand flow control in your program.

This is why we like type systems: they remind you to check!
So here, we check in ways that were already idiomatic in the language,
and the type system will understand that type guard. `state ~== null`

What is Refinement Invalidation?

If we change the state, it could go back to null.

TS won't error here. We can solve that by pulling it out into a constant here.

Here is where TS and Flow start diverging!
Flow leans toward soundness:
if you call anything that could possibly change the value,
it's an error.
Basically forces you to follow the pattern of pulling it out
into a local variable (or constant).

Why didn't TS do this?
Optimism: this is already a good practice,
and often calls like this are e.g. logging anyway.

Also: hard to produce good error messages!!

#### Sorbet

run time!
can start with logging, then turn on for real to fail.

#### Statically Typed

They can bake it in from the beginning.

#### recap

## Case study 3: Variance

This one is a bit deeper: a concept called variance.

Array of `Shape` vs array of `Square`

TS will not catch this; Flow will.
If something wants an array of shapes, you need to pass it an array of shapes.

Variance describes rules for _subtyping_.

There are four ways we can handle variance.

Covariance is the one that makes sense to most people, at first.
Contravariance doesn't make sense at first, and makes no sense for arrays,
but we'll see where it makes sense.
Invariance: simplest possible thing.
Bivariance: we'll let you do either one, and assume you have a handle on it.

Not just arrays: functions are another higher-order type,
that return a type of thing.

Sometimes there are straightforward correct answers: function examples.
This may not make sense at first! We'll see some examples.

Producers are covariant. Why?
Goes back to the idea of substitutability.
If we need some music, and as a general entertainer, we might get juggling!

We can think of functions as producers of their return type.

Contravariance:

Now let's talk about consumers of energy sources.

A big ball of fire can definitely consume an artichoke,
but a panda can't.

Functions *consume* their arguments, so we need contravariance there.

#### Arrays

There are things you can do with an array of Shapes,
that you can't do with an arrao of Squares: add a circle.
Covariance, not sound.

Contravariant? No, because getting an element from an array
of squares, definitely a square; array of shapes, nope.
Also not sound!

Bivariance, for sure not sound: both others not found.

Invariance: sound.

So why is TS covariant? Optimism and completeness.
We're usually using them as producers, it's OK.
it is *more complete* but *less sound*.

Flow enforces invariance, but gives escape hatch: `$ReadOnlyArray`,
where you can use it only as a producer.

### MyPy

For lists, invariance.
But Pyhone already has read-only `Sequence` as a built-in concept!

### Java

Java is unsound here: Can pass in `String[]` if `Shape[]` expected.
Trade-off is, it has to do a run-time type check on write.
Then run-time type error.

### Go

Different lang philosophy: as simple as possible -> inviariance.

### Rust

built-in concept of mutability.

### recap

Some languages assume read-only is usual case, let you do it.

Go goes for simplicity: clear error messages, definitely sound.

Rust makes it explicit.

## recap after case studies

Soundness isn't the only goal.
Completeness
easy to learn
great error messages

Completeness isn't the primary goal.

Feels more complex than in java/go/etc?
Need to work with existing idiom (e.g. run-time type checks)
and features.

There is often no right answer.
Depends on type system's goals.

# Conclusion

What I wish I'd known a few years ago:
When picking a type system, you can ask:
how does the team approach these trade-offs,
and does that approach match your team's priorities.

Are you sending a rocket to space? If so, you may prioritize soundness.
In other situations, you may just want something that makes you a little more productive.



<!-- Note on images
  Images (e.g. my_image.jpg) should be put in the `website/static/blog/strange-loop-2019` directory, with the path to the image in your post being `/blog/strange-loop-2019/my_image.jpg`. If you'd rather host the images somewhere else for ease of use, that's fine too.

  Please also try to keep your images to a reasonable size by:
    - Using JPEG compression, unless image is mostly solid color 
    - JPEG compression set between 60%-80%
    - Resizing the image to be no wider then 750px
    - If PNG, use a tool like ImageOptim (https://imageoptim.com/mac) to optimize the file size

  I suggest re-sizing and compressing all the images in one batch as a last step.
-->  
