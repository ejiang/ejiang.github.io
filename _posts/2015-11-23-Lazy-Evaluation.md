---
layout: post
title:  "Lazy Evaluation"
date:   2015-11-23 20:40:00
categories: cs
---

I attempt an explanation for lazy evaluation. Bold words are key concepts and are
bolded every time mentioned for clarity.

First, we take a look at the lazy evaluator to understand what occurs.
This assumes an understanding of MCE.

## From 10000 feet

At its heart, a Scheme interpreter is just doing a huge **expansion** (ignore set! and such for
now). We have procedures that we define, but they can just be replaced by their
body whenever they are called.

For instance,

{% highlight scheme %}
(define (foo x)
  (define (bar y)
    (+ x y))
  bar)
((foo 12) 12)
(bar 12)
(+ 12 12)
; at the end of the day you go all the way down to the primitive expression
{% endhighlight %}

Let's take a step back and think about what kind of characters we meet here.

We can roughly divide into two groups: single items (such as
the number 1, the variable foo, the symbol 'bar, all the things on mc-eval), and
expression types (define, if/cond, application, set!, lambdas (which essentially
just contain delayed other expressions)).

With regards to the mentioned expression types, application is really the only one
that is expanded. We have the compound procedure, that is, the ones you're used to, created with
define and containing some body.

Now, we could just keep replacing every compound procedure call with its body (call this 
**expansion**), but
that would lead to infinite expansion when you have recursion (think about factorial).
We have special forms, like if. If then, can be thought of as controlling
the **expansion** of procedure calls.

Also, let's take note of the role of the environment here. Note that in the code example,
you had to keep track of the x. That is, if we just saw (bar 12) we don't have all the 
information needed. However, at the end, the environment merely assigns values to names.
Once we've reached (+ 12 12) we don't care how it got there.

What lazy evaluation notices is that often times there may be arguments passed that don't
"make it" to the final, **distilled expression** (which can be evaluated immediately
to obtain the final value).

Lazy evaluation, when we don't have assignment, gives the exact same values as regular
evaluation, except perhaps even better because we can sometimes avoid evaluating (/ 1 0) and
other bad expressions. When assignment comes into the picture, it doesn't do the same thing.

## Lazy Evaluation

So with our model in account, we take a look at the three instances when lazy forces
a thunk.

 1. when it is passed to a primitive procedure that will use the value of the thunk
 2. when it is the value of a predicate of a conditional
 3. when it is the value of an operator that is about to be applied as a procedure

Let's take a look at some motivating examples.

### Example 1

1. when it is passed to a primitive procedure that will use the value of the thunk

{% highlight scheme %}
(define (foo a b c)
  (+ b c)
  (+ a b))
(foo 1 2 (/ 1 0))
; error
{% endhighlight %}

What? According to what was stated, shouldn't this have just kept (+ a b) and returned 3?

Well, (+ a b) may be the last value and therefore the value of the entire expression, but
when we say, we want the **distilled expression** to be the same, we must note what
that distilled expression would look like. In this case, it is actually.

{% highlight scheme %}
(define (foo a b c)
  (+ b c)
  (+ a b))
(foo 1 2 (/ 1 0))

(+ 2 (/ 1 0)) ; these last two lines
(+ 1 2)
{% endhighlight %}

Think of it as though you had typed it into the interactive prompt, and what the output
would be.

### Example 2

2. when it is the value of a predicate of a conditional

{% highlight scheme %}
(define (foo x y z)
  (if x
    12
    z))
(foo #t (/ 1 0) (/ 1 0))
12
{% endhighlight %}

We have the conditionals/ifs controlling **expansion**. This expansion is really the
core of the program: how it flows, and when it ends. Notice how the recursive procedures we
write in CS61AS follow the same pattern of conds, base cases, and recursive calls. Essentially,
it's a matter of which cond we go into that is the heart of every program.

Notice that #t, the value of x, never appears in the **distilled expression** yet it
determines what it will be. In other words, x's value matters, but it doesn't make its
way into the **distilled expression**.

### Example 3

3. when it is the value of an operator that is about to be applied as a procedure

{% highlight scheme %}
(define (foo a b c d e f)
  (define (bar a b c d e f)
    (+ a 12))
  (bar a b c d e f))
(foo 1 (/ 1 0) (/ 1 0) (/ 1 0) (/ 1 0) (/ 1 0))
(bar 1 (/ 1 0) (/ 1 0) (/ 1 0) (/ 1 0) (/ 1 0))
(+ 1 12)
13
{% endhighlight %}

What about a procedure call within a procedure, as we have here? Certainly we don't want
to evaluate b, c, d, e, and f.

Note that it says we must get the *value of the operator, not the values of the arguments to the operator*. Operator in this case refers to any procedure, primitive or compound.

If we want to get 13 as the **distilled expression**, we can't just leave it at
(bar 1 (/ 1 0) (/ 1 0) (/ 1 0) (/ 1 0) (/ 1 0)). We have to perform an **expansion** of bar
and put in bar's body.

### Wrapping up the examples

So now we see that the **distilled expression** or final value rather
is determined by many things.

 1. The actual primitive operators like + that do the computation with the values
 2. The flow through the conds/ifs and which then-statements get expanded. Note this does
    not reflect in values to the **distilled expression** themselves
    the but is crucial to knowing which **distilled expression** we get.
 3. The contents of the compound procedures that we have to go through so that we can
    get to the primitive operators at the end

Let's take look at one final example to combine everything we've learned.

{% highlight scheme %}
(define (foo a b c d e)
  (define (bar x)
    (+ x a))
  (if b
    (bar c)
    d))
{% endhighlight %}

What are a, b, c, d, and e here?

 - a fits rule 1
 - b fits rule 2
 - bar fits rule 3
 - c could fit rule 1 also, depending on value of b
 - d could fit rule 1 also, depending on value of b
 - e never gets evaluated because it fits none of the rules

## Assignment

What about set! ? I think one confusion people have with set is similar to the
confusion raised in example 1, that because it is not the last value it doesn't
really matter, that it does not make it to the **distilled expression**. However,
this is based on an incorrect understanding of the **distilled expression**. It includes
things like the first (+ 2 (/ 1 0)). All primitive calls are part of the **distilled expression**, not just the last ones.

Can we think of an instance where set! would work different in regular and in lazy?

{% highlight scheme %}
(define c 0)
(define (bar x)
  (set! c x)
  x)
(define (foo y z)
  y)
(foo 12 (bar 34))
{% endhighlight %}

In regular, c would be 34. In lazy, it would not be. However, set! isn't really special
compared to + and other primitives. This is an instance of when set! is part of (bar 34)
which doesn't fit under any of the 3 rules.

## Speed

One of the things we want to know, besides what will happen, is how fast compared to other
interpreters. Lazy does better if you have some slow arguments to a procedure that don't
follow any of the 3 rules. Analyzing does better if you have many recursive calls because
lazy still follows the regular parsing of mceval.

## Practical Tips

 - Count applications of the rules if you're confused regarding set!s. That is,
   you can keep tabs on "rule 1 is applied twice, rule 2 is applied once" etc.

## Sources

 - SICP
 - cs61as.org

