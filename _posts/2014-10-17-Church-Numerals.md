---
layout: post
title:  "Church Numerals"
date:   2015-10-17 03:25:51
categories: cs
---
Church numerals appear in SICP 2.6. Here I attempt an explanation.

## Original

{% highlight scheme %}
(define zero (lambda (f) (lambda (x) x)))

(define (add-1 n)
  (lambda (f) (lambda (x) (f ((n f) x)))))
{% endhighlight %}

## Explanation

Church numerals are a way of encoding the natural numbers into lambdas. It's an alternative
representation of the natural numbers that doesn't rely on having them defined by the programming
language. We can use Church encoding to see the power of lambdas.

## Zero

Numbers in this system are represented as how many times a function f is applied. So zero then
is a function such that:

 - input: some one argument f
 - output: some function that takes in an x and outputs x (so whatever f you pass in doesn't matter,
    since it's applied 0 times)

So what is one then, in this system? Following the logic of zero, one would be a function such that:

 - input: some one argument f
 - output: some function that takes in x and outputs (f x)

In code, then:

{% highlight scheme %}
(define one
  (lambda (f)
    (lambda (x)
      (f x))))
{% endhighlight %}

Two, then follows:

{% highlight scheme %}
(define two
  (lambda (f)
    (lambda (y)
      (f (f y)))))
{% endhighlight %}

Three, for the next part:

{% highlight scheme %}
(define three
  (lambda (f)
    (lambda (z)
      (f (f (f z))))))
{% endhighlight %}

However, we can't just type this out explicitly for every single number. We want to add together numbers.
But we can't just do (two (three f)) to add, because that gives us:

{% highlight scheme %}
(two (three f))

(lambda (x) ((three f) ((three f) x)))

(lambda (x) (three (f (f (f x)))))
(lambda (x) (f (f (f (f (f (f x)))))))

{% endhighlight %}

So we would end up multiplying instead of adding.

So we take a look at add-1 (as defined above). The result of (add-1 n) takes in a number x and applies f one more time
after applying f n times on x. Recall that (n f) is the equivalent of f^(n) in mathematics. So in
more familiar notation, f(f^n(x)) = f^(n+1)(x).

Now the final question asks, we want a definition of addition in this system. We want to add
m and n. So in familiar notation f^m(f^n(x)) = f^(m+n)(x).

So, we have a generic add:

{% highlight scheme %}

(define (add m n)
  (lambda (f)
    (lambda (x)
      ((m f) ((n f) x)))))

(((add two three) (lambda (x) (+ 1 x))) 0)
; 5

{% endhighlight %}
