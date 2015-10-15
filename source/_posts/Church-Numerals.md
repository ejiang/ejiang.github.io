title: Church Numerals
date: 2015-10-14 18:16:13
tags: church_numeral
---

Church numerals appear in SICP 2.6. Here I attempt an explanation.

## Original

{% codeblock 2.6 lang:scheme https://mitpress.mit.edu/sicp/full-text/book/book-Z-H-14.html#%_thm_2.6 %}
(define zero (lambda (f) (lambda (x) x)))

(define (add-1 n)
  (lambda (f) (lambda (x) (f ((n f) x)))))
{% endcodeblock %}

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

 - input: some argument f
 - output: some function that takes in x and outputs (f x)

In code, then:

{% codeblock one %}
(define one
  (lambda (f)
    (lambda (x)
      (f x))))
{% endcodeblock %}

Two, then follows:

{% codeblock two %}
(define two
  (lambda (f)
    (lambda (y)
      (f (f y)))))
{% endcodeblock %}

Three, for the next part:

{% codeblock three %}
(define three
  (lambda (f)
    (lambda (z)
      (f (f (f z))))))
{% endcodeblock %}

However, we can't just type this for every single number. We want to add together numbers.
But we can't just do (two three) to add, because that gives us:

{% codeblock three %}
(two three)

(lambda (x) (three (three x)))

(lambda (x) (three (lambda (z) (x (x (x z)))))) ; evaluate innermost first

{% endcodeblock %}

We run into a type error first of all, but even if we could somehow compose them, we would actually
be getting 6 applications instead of 5, because we would be multiplying rather than adding.

So we take a look at add-1. The result of (add-1 n) takes in a number x and applies f one more time
after applying f n times on x. Recall that (n f) is the equivalent of f^(n) in mathematics. So in
more familiar notation, f(f^n(x)) = f^(n+1)(x).

Now the final question asks, we want a definition of addition in this system. We want to add
m and n. So in familiar notation f^m(f^n(x)) = f^(m+n)(x).

So, we have a generic add:

{% codeblock add %}

(define (add m n)
  (lambda (f)
    (lambda (x)
      ((m f) ((n f) x)))))

(((add two three) (lambda (x) (+ 1 x))) 0)
; 5

{% endcodeblock %}
