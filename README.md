# The Bel Language

This is a mirror of Paul Graham's [Bel language](http://paulgraham.com/bel.html)
web page, converted to Markdown and provided with syntax highlighting
for ease of reading.

- Below is the text of the [language guide](https://sep.yimg.com/ty/cdn/paulgraham/bellanguage.txt?t=1570993483&) with Markdown mark-up.
- [This](./bel.bel.lisp) is Bel's [source code](https://sep.yimg.com/ty/cdn/paulgraham/bel.bel?t=1570993483&) (`.lisp` extension is added for syntax highlighting);
- [This](./examples.md) is the file of Bel [code examples](https://sep.yimg.com/ty/cdn/paulgraham/belexamples.txt?t=1570993483&).

## The Bel Language guide
12 Oct 2019


In 1960 John McCarthy described a new type of programming language
called Lisp. I say "new type" because Lisp represented not just a new
language but a new way of describing languages. He defined Lisp by 
starting with a small set of operators, akin to axioms, and then 
using them to write an interpreter for the language in itself.

His goal was not to define a programming language in our sense of
the word: a language used to tell computers what to do. The Lisp
in his 1960 paper was meant to be a formal model of computation,
like a Turing Machine. McCarthy didn't realize it could be used on
computers till his graduate student Steve Russell suggested it.

The Lisp in the 1960 paper was missing features you'd need in a 
programming language. There were no numbers, for example, or errors, 
or I/O. So when people used it as the basis for languages used to 
program computers, such things had to be added. And when they were, 
the axiomatic approach was dropped.

So the development of Lisp happened in two parts (though they seem 
to have been interleaved somewhat): a formal phase, represented by 
the 1960 paper, and an implementation phase, in which this language
was adapted and extended to run on computers. Most of the work, as
measured by features, took place in the implementation phase. The
Lisp in the 1960 paper, translated into Common Lisp, is only 53 lines 
of code. It does only as much as it has to in order to interpret 
expressions. Everything else got added in the implementation phase.

My hypothesis is that, though an accident of history, it was a good 
thing for Lisp that its development happened in two phases-- that
the initial exercise of defining the language by writing an 
interpreter for it in itself is responsible for a lot of Lisp's best 
qualities. And if so, why not do more of it?

Bel is an attempt to answer the question: what happens if, instead of
switching from the formal to the implementation phase as soon as 
possible, you try to delay that switch for as long as possible? If 
you keep using the axiomatic approach till you have something close 
to a complete programming language, what axioms do you need, and what
does the resulting language look like?

I want to be clear about what Bel is and isn't. Although it has a lot
more features than McCarthy's 1960 Lisp, it's still only the product
of the formal phase. This is not a language you can use to program 
computers, just as the Lisp in the 1960 paper wasn't. Mainly because, 
like McCarthy's Lisp, it is not at all concerned with efficiency. 
When I define append in Bel, I'm saying what append means, not trying 
to provide an efficient implementation of it.

Why do this? Why prolong the formal phase? One answer is that it's
an interesting exercise in itself to see where the axiomatic approach 
leads. If computers were as powerful as we wanted, what would 
languages look like?

But I also believe that it will be possible to write efficient
implementations based on Bel, by adding restrictions. If you want a 
language with expressive power, clarity, and efficiency, it may work
better to start with expressive power and clarity, and then add 
restrictions, than to approach from another direction.

So if you'd like to try writing an implementation based on Bel, 
please do. I'll be one of your first users.

I've ended up reproducing a number of things in previous dialects.
Either their designers got it right, or I'm too influenced by
dialects I've used to see the right answer; time will tell. I've also 
tried to avoid gratuitous departures from existing Lisp conventions. 
Which means if you see a departure from existing conventions, there 
is probably a reason for it.



## Data

Bel has four fundamental data types: symbols, pairs, characters, and
streams.

Symbols are words:

```lisp
foo
```

The names of symbols are case-sensitive, so foo and Foo are distinct
symbols.

Pairs are pairs of any two things, and are represented thus:

```lisp
(foo . bar)
```

That's a pair of two symbols, foo and bar, but the two halves of a
pair can be anything, including pairs:

```lisp
(foo . (bar . baz))
```

That's a pair of the symbol foo, and the pair `(bar . baz)`.

A character is represented by prepending a backslash to its name. So 
the letter `a` is represented as

```lisp
\a
```

Characters that aren't letters may have longer names. For example the
bell character, after which Bel is named, is

```lisp
\bel
```

There is no way of representing a stream. If Bel has to display a
stream, it prints something that will cause an error if it's read
back in.

Anything that's not a pair is called an atom. So symbols, characters,
and streams are atoms.

Instances of the four fundamental types are called objects.


## Lists

We can use pairs to build lots of different data structures, but the 
most fundamental way they're used is to make lists, as follows:

1. The symbol nil represents the empty list.

2. If `y` is a list, then the pair `(x . y)` is a list of `x` followed by 
   the elements of `y`.

Here's a list made from a pair:

```lisp
(a . nil)
```

According to rule 2, this is a list of the symbol a followed by the
elements of nil, of which according to rule 1 there are none. So it 
is a list of one element, the symbol `a`.

By nesting such pairs we can create lists of any length. Here is a
list of two elements, the symbols `a` and `b`:

```lisp
(a . (b . nil))
```

And here is a list of a, b, and c:

```lisp
(a . (b . (c . nil)))
```

This would be an awkward way to express lists, but there is an
abbreviated notation that's more convenient:

1. The symbol `nil` can also be represented as `()`.

2. When the second half of a pair is a list, you can omit the dot 
   before it and the parentheses around it. So `(a . (b ...))` can 
   be written as `(a b ...)`.

By repeated application of these two rules we can transform

```lisp
(a . (b . (c . nil)))
```

into

```lisp
(a b c)
```

In other words, a list can be expressed as its elements within
parentheses. You wouldn't use dot notation for a list like `(a b c)`
unless there was some special reason to.

Because any object can be part of a pair, the elements of lists can
themselves be lists. All these are lists too:

```lisp
(a (b) c)
((a b c))
(nil)
```

Pairs like these, where if you keep looking at the second half you
eventually hit a nil, are called proper lists. This is a proper
list:

```lisp
(a b c)
```

and this is not:

```lisp
(a b . c)
```

The empty list is also a proper list.

A pair that's not a proper list is called a dotted list (because
you need to use dot notation to represent it).

A proper list of characters is called a string, and can also be
represented as those characters within double-quotes. So the list

```lisp
(\h \e \l \l \o)
```

can also be represented as

```lisp
"hello"
```

and will when possible be displayed that way.


## Truth

The symbol `nil` represents falsity as well as the empty list. The 
symbol `t` is the default representation for truth, but any object
other than nil also counts as true. 

It may seem strange to use the same value to represent both falsity
and the empty list, but in practice it works well. Lisp functions
often return sets of answers, and the empty set of answers is 
falsity.


## Functions

Bel programs consist mostly of functions. Functions take zero or more
objects as arguments, perhaps do something (e.g. print a message), 
and return one object.

Functions are represented using lists. For example, here is a
function that takes one argument, and returns that plus 1.

```lisp
(lit clo nil (x) (+ x 1))
```

The first element, `lit`, says that this is a literal object, not to be 
evaluated.

The second, `clo`, says what kind of literal it is: a closure.

The third is the local enviroment, a list of variables that already
have values from having been parameters of functions. This example 
has an empty environment.

The fourth, `(x)`, is the function's parameters. When the function is 
called, the value of `x` will be whatever it was called with.

The fifth and last element, `(+ x 1)`, defines the value that the
function returns.

You would not ordinarily express a function using its literal 
representation. Usually you'd say

```lisp
(fn (x) (+ x 1))
```

which yields the function above.


## Evaluation

The execution of a Bel program consists of the evaluation of
expressions. All Bel objects are expressions, so the word "expression"
is merely a statement of intention: it means an object that you
expect to be evaluated.

When an expression is evaluated, there are three possible outcomes:

1. It can return a value: `(+ 1 2)` returns `3`.

2. It can cause an error: `(/ 1 0)` will.

3. It can fail to terminate: `(while t)` will.

Some expressions also do things in the process of being evaluated.
For example,

```lisp
(prn 1)
```

will return 1, but before doing so will print it.

Some atoms evaluate to themselves. All characters and streams do,
along with the symbols `nil`, `t`, `o`, and `apply`. All other symbols
are variable names, and either evaluate to some value, or cause an
error if they don't have a value.

A proper list whose first element evaluates to a function is called
a function call. For example, the expression

```lisp
(+ x 1)
```

is a function call, because the value of `+` is a function. The value 
of a function call is the object that the function returns.

Function calls are evaluated left to right. For example, when

```lisp
(+ 8 5)
```

is evaluated,

1. First `+` is evaluated, returning a function that returns the sum
   of its arguments.

2. Then `8` is evaluated, returning itself.

3. Then `5` is evaluated, also returning itself.

4. Finally, the two numbers are passed to the function, which returns
   `13`.

If we want to show what expressions evaluate to, it's conventional to 
show them being evaluated in a repl:

```lisp
> (+ 8 5)
13
```

Expressions can be nested. The rule of evaluating left to right means 
that nested function calls are evaluated depth-first. For example, 
when

```lisp
(+ (- 5 2) 7)
```

is evaluated, the subexpressions that get evaluated are, in order,
`+`, `(- 5 2)`, `-`, `5`, `2`, `7`.

Not all expressions are evaluated left to right, however. There is a 
small set of symbols called special forms, and an expression whose
first element is a special form is evaluated according to rules
defined for that special form.

For example, if is a special form, and when an expression of the form

```lisp
(if test then else)
```

is evaluated, only one of the last two elements will be evaluated,
depending on whether the second element, test, returns true or not.

Things meant to be used as the first element of an expression are
called operators. So functions and special forms are operators. But
like the term "expression," this is just a statement of intention.
You can put anything first in an expression so long as you specify 
what happens when you do.


## Bindings and Environments

There are three ways a variable can have a value. It can have a value 
globally, as for example + does, meaning that by default it has this 
value everywhere. Such a variable is said to be globally bound, and 
the set of global bindings is called the global environment.

Another way a variable can have a value is by being a parameter in a 
function. When the function

```lisp
(fn (x) (+ x 1))
```

is called, the variable `x` will, within it, have as its value whatever 
argument the function was called with. That's called a lexical 
binding, and the current set of lexical bindings is the lexical 
environment. 

Finally, variables can have dynamic bindings, which are visible
everywhere, like global bindings, but temporary: they persist only
during the evaluation of whatever expression created them.

Dynamic bindings take precendence over lexical bindings, which take 
precedence over global ones.

If you do an assignment to a variable that has one of the three kinds 
of bindings, you'll modify whichever binding is currently visible. If 
you do an assignment to a variable that's not bound, you'll create a 
global binding for it.


## Errors

Errors are signalled by calling `err` with one argument describing the 
error. Bel doesn't specify a global binding for `err`; this is 
something for a programming environment built on top of Bel to do. 
But some error-catching code in the Bel source does dynamically bind 
`err`.


## Axioms

Like McCarthy's Lisp, Bel is defined starting with a set of operators
that we have to assume already exist. Then more are defined in terms 
of these, till finally we can define a function that is a Bel 
interpreter, meaning a function that takes a Bel expression as an 
argument and evaluates it.

There are two main types of axioms: primitives and special forms.
There are also a few variables that come predefined.

In the following sections, the descriptions of primitives are their
definitions. The descriptions of special forms, however, are merely
summaries of their behavior; special forms are defined by the code
that implements them in the Bel source.


Variables and Constants

1. `t nil o apply`

Evaluate to themselves.

2. `chars`

A list of all characters. Its elements are of the form `(c . b)`, where
`c` is a character and `b` is its binary representation in the form of a 
string of `\1` and `\0` characters. Bel doesn't specify which characters
are in chars, but obviously they should include at least those in the 
Bel source.

3. `globe scope`

The current global and lexical environments, represented as lists of 
`(var . val)` pairs of variables and their values.

4. `ins outs`

The default input and output streams. Initially `nil`, which represents
the initial input and output streams. Bel doesn't specify what those
are, but if you started Bel at a prompt you'd expect them to be the
terminal.


## Quote

The quote operator is a special form, but it has to be described 
first because so many code examples use it.

It returns its argument without evaluating it. Its purpose is to 
prevent evaluation.

```lisp
> (quote a)
a
```

Prepending `'` to an expression is equivalent to wrapping a quote
around it.

```lisp
> 'a
a
```

Why do you need to prevent evaluation? To distinguish between code
and data. If you want to talk about the symbol `a`, you have to quote
it. Otherwise it will be treated as a variable, and you'll get its
value. E.g. if `a` has been set to 10:

```lisp
> a
10
> 'a
a
```

Because the symbols `nil`, `t`, `o`, and `apply` evaluate to themselves, you 
don't have to quote them. Ditto for strings.


## Primitives

Primitives can be called like functions, but are assumed to exist, 
rather than defined in the Bel source. As with function calls, the
arguments in calls to primitives are all evaluated, left to right. 
Missing arguments default to nil. Extra arguments cause an error to 
be signalled.


### 1.  `(id x y)`

Returns true iff x and y are identical. 

```lisp
> (id 'a 'a)
t
> (id 'a 'b)
nil
```

Identity is stricter than equality. While there is only one of each
symbol and character, there can be any number of different pairs with 
the same elements. So two pairs can look the same without being
identical:

```lisp
> (id '(a b) '(a b))
nil
```

Because `id` is so strict, it's not the function you'd usually use to 
test for equality. Usually you'd use `=`.


### 2.  `(join x y)`

Returns a new pair whose first half is `x` and second half is `y`.

```lisp
> (join 'a 'b)
(a . b)
> (join 'a)
(a)
```

A pair returned by `join` will not be `id` to any existing pair.

```lisp
> (id (join 'a 'b) (join 'a 'b))
nil
```


### 3.  `(car x)`

Returns the first half of a pair:

```lisp
> (car '(a . b))
a
> (car '(a b))
a
```

The `car` of `nil` is `nil`,

```lisp
> (car nil)
nil
```

but calling `car` on any atom other than `nil` will cause an error.

The name "car" is McCarthy's. It's a reference to the architecture of 
the first computer Lisp ran on. But though the name is a historical 
accident, it works so well in practice that there's no reason to 
change it.


### 4.  `(cdr x)`

Returns the second half of a pair:

```lisp
> (cdr '(a . b))
b
> (cdr '(a b))
(b)
```

As with `car`, calling it on `nil` yields `nil`, calling it on any other
atom causes an error, and the name is McCarthy's.

When operating on pairs used to represent lists, `car` and `cdr` get you 
the first element and the rest of the list respectively.


### 5.  `(type x)`

Returns either symbol, pair, char, or stream depending on the type
of `x`.

```lisp
> (type 'a)
symbol
> (type '(a))
pair
> (type \a)
char
```


### 6.  `(xar x y)`

Replaces the `car` of `x` with `y`, returning `y`. Signals an error if `x` is 
not a pair.

If we assume that the value of `x` is `(a . b)`, then

```lisp
> x
(a . b)
> (xar x 'c)
c
> x
(c . b)
```


### 7.  `(xdr x y)`

Like `xar`, except that it replaces the `cdr` of `x`.

```lisp
> x
(c . b)
> (xdr x 'd)
d
> x
(c . d)
```


### 8.  `(sym x)`

Returns the symbol whose name is the elements of `x`. Signals an error
if `x` is not a string.

```lisp
> (sym "foo")
foo
```


### 9.  `(nom x)`

Returns a fresh list of the characters in the name of `x`. Signals an 
error if `x` is not a symbol.

```lisp
> (nom 'foo)
"foo"
```


### 10. `(wrb x y)`

Writes the bit `x` (represented by either `\1` or `\0`) to the stream `y`. 
Returns `x`. Signals an error if it can't or if `x` is not `\1` or `\0`. If `y` 
is `nil`, writes to the initial output stream. 


### 11. `(rdb x)`

Tries to read a bit from the stream `x`. Returns `\1` or `\0` if it finds 
one, `nil` if no bit is currently available, or `eof` if no more will be 
available. Signals an error if it can't. If `x` is `nil`, reads from the 
initial input stream.


### 12. `(ops x y)`

Returns a stream that writes to or reads from the place whose name is 
the string `x`, depending on whether `y` is `out` or `in` respectively. 
Signals an error if it can't, or if `y` is not `out` or `in`.


### 13. `(cls x)`

Closes the stream `x`. Signals an error if it can't.


### 14. `(stat x)`

Returns either `closed`, `in`, or `out` depending on whether the stream `x` 
is closed, or reading from or writing to something respectively. 
Signals an error if it can't.


### 15. `(coin)`

Returns either `t` or `nil` randomly.


### 16. `(sys x)`

Sends `x` as a command to the operating system. 


## Special Forms

Expressions beginning with special forms are not always evaluated in 
the usual left-to-right way.


### 1.  `(quote x)`

Described above.


### 2.  `(lit ...)`

Returns the whole lit expression without evaluating it. A lit is like
a persistent quote; evaluation strips the quote off a quote 
expression, but leaves a lit expression intact.

```lisp
> (quote a)
a
> (lit a)
(lit a)
```

The name stands for literal, and it can take any number of arguments.
This is how you make things that evaluate to themselves, the way
characters or `nil` do. Functions are lits, for example, as are
numbers.

The value of a primitive `p` is `(lit prim p)`. 

```lisp
> car
(lit prim car)
```


### 3.  `(if ...)`

An if expression with an odd number of arguments

```lisp
(if a1 a2 a3 a4 ... an)
```

is equivalent to

> if a1 then a2 else if a3 then a4 ... else an

I.e. the odd numbered arguments are evaluated in order till we either
reach the last, or one returns true.  In the former case, its value
is returned as the value of the if expression. In the latter, the
succeeding argument is evaluated and its value returned.

An if expression with an even number of arguments

```lisp
(if a1 a2 ... an)
```

is equivalent to

```lisp
(if a1 a2 ... an nil)
```

Falsity is represented by `nil`, and truth by any other value. Tests 
generally return the symbol `t` when they can't return anything more 
useful.

As a rule I've tried to make axioms as weak as possible. But while
a 3-argument if would have sufficed, an n-argument version didn't
require significantly more code, so it seemed gratuitously fussy to 
insist on 3 arguments.


### 4.  `(apply f ...)`

An expression of the form

```lisp
(apply f x y ... z)
```

is equivalent to

```lisp
(f 'a 'b ... 'c1 ... 'cn)
```

where `a` is the value of `x`, `b` the value of `y`, and the `ci` the elements 
of the value of `z`.

```lisp
> (join 'a 'b)
(a . b)
> (apply join '(a b))
(a . b)
> (apply join 'a '(b))
(a . b)
```

The last argument to apply can be a dotted list if it matches the 
parameters of the first.


### 5.  `(where x)`

Evaluates `x`. If its value comes from a pair, returns a list of that
pair and either `a` or `d` depending on whether the value is stored in
the car or cdr. Signals an error if the value of `x` doesn't come from 
a pair.

For example, if `x` is `(a b c)`,

```lisp
> (where (cdr x))
((a b c) d)
```


### 6.  `(dyn v x y)`

Evaluates `x`, then causes `y` to be evaluated with the variable `v`
dynamically bound to the value of `x`. 

For example, if `x` is `a`,

```lisp
> x
a
> (dyn x 'z (join x 'b))
(z . b)
> x
a
```


### 7.  `(after x y)`

Evaluates both its arguments in order. The second will be evaluated
even if the evaluation of the first is interrupted (e.g. by an
error). 


### 8.  `(ccc f)`

Evaluates `f` and calls its value on the current continuation. The
continuation, if called with one argument, will return it as the 
value of the `ccc` expression (even if you are no longer in the `ccc` 
expression or in code called by it).


### 9.  `(thread x)`

Starts a new thread in which `x` will be evaluated. Global bindings are 
shared between threads, but not dynamic ones.


## Reading the Source

Starting with the foregoing 25 operators, we're going to define more,
till we can define a Bel interpreter. Then we'll continue, defining
numbers, I/O, and several other things one needs in programs.

These definition are in the Bel source, which is meant to be read in
parallel with this guide.

In the Bel source, when you see an expression of the form 

```lisp
(set v1 e1 ... vn en)
```

it means each `vi` is globally bound to the value of `ei`.

In the source I try not to use things before I've defined them, but 
I've made a handful of exceptions to make the code easier to read.

When you see

```lisp
(def n p e)
```

treat it as an abbreviation for 

```lisp
(set n (lit clo nil p e))
```

and when you see 

```lisp
(mac n p e)
```

treat it as an abbreviation for

```lisp
(set n (lit mac (lit clo nil p e)))
```

The actual `def` and `mac` operators are more powerful, but this is as 
much as we need to start with.

Treat an expression in square brackets, e.g.

```lisp
[f _ x]
```

as an abbreviation for 

```lisp
(fn (_) (f _ x))
```

In Bel, underscore is an ordinary character and `_` is thus an ordinary 
variable.

Finally, treat an expression with a prepended backquote (`) as a
quoted list, but with "holes," marked by commas, where evaluation 
is turned back on again. 

```lisp
> (set x 'a)
a
> `(x ,x y)
(x a y)
> `(x ,x y ,(+ 1 2))
(x a y 3)
```

You can also use `,@` to get a value spliced into the surrounding
list:

```lisp
> (set y '(c d))
(c d)
> `(a b ,@y e f)
(a b c d e f)
```

Now let's look at the source. The first expression defines a function
no that takes one argument, `x`, and returns the result of using id to
compare it to nil. So `no` returns `t` if its argument is nil, and nil 
otherwise.

```lisp
> (no nil)
t
> (no 'a)
nil
```

Since `nil` represents both falsity and the empty list, no is both
logical negation and the test for the empty list. 

The second function, `atom`, returns true iff its argument is not a 
pair. 

```lisp
> (atom \a)
t
> (atom nil)
t
> (atom 'a)
t
> (atom '(a))
nil
```

Next come a pair of similar functions, `all` and `some`. The former 
returns `t` iff its first argument returns true of all the elements of 
its second,

```lisp
> (all atom '(a b))
t
> (all atom nil)
t
> (all atom '(a (b c) d))
nil
```

and the latter returns true iff its first argument returns true of 
any element of its second. However, when `some` returns true, it 
doesn't simply return `t`. It returns the remainder of the list 
starting from the point where `f` was true of the first element. 

```lisp
> (some atom '((a b) (c d)))
nil
> (some atom '((a b) c (d e)))
(c (d e))
```

Logically, any value except `nil` counts as truth, so why not return 
the most informative result you can?

In `all` and some we see the first use of `if`. Translated into English, 
the definition of all might be:

>  If xs is empty, then return t. 
>
>  Otherwise if f returns true of the first element, return the result 
>  of calling all on f and the remaining elements. 
>
>  Otherwise (in which case xs has at least one element of which f
>  returns false), return nil.

This technique of doing something to the `car` of a list and then 
perhaps continuing down the cdr is very common.

Something else is new in `all` and `some`: these are the first functions
in the Bel source that you could cause an error by calling.

```lisp
> (all atom 'a)
Error: can't call car on a non-nil atom.
```

I made up that error message; Bel doesn't specify more about errors
in primitives than when they occur, and doesn't specify anything 
about repls. But some error will be signalled if you call all with a
non-nil atom as the second argument, because in the second test 
within the `if`

```lisp
(f (car xs))
```

`car` is called on it, and it's an error to call `car` on anything except
a pair or `nil`.

One other thing to note about these definitions, now that we're
getting to more complex ones: these functions are not defined the
way they would be in idiomatic Bel. For example, if all didn't 
already exist in Bel you could define it as simply

```lisp
(def all (f xs)
  (~some ~f xs))
```

But since we haven't defined functional composition yet, I didn't use 
it.

The next function, `reduce`, is for combining the elements of its 
second argument using nested calls to its first. For example 

```lisp
(reduce f '(a b c d))
```

is equivalent to

```lisp
(f 'a (f 'b (f 'c 'd)))
```

If `xs` has only one element, `reduce` returns it, and if it's empty,
reduce returns nil; since `(cdr nil)` is nil, we can check both these 
possibilities with `(no (cdr xs))`. Otherwise it calls `f` on the first
element and `reduce` of `f` and the remaining elements.

```lisp
> (reduce join '(a b c))
(a b . c)
```

This is not the only way to reduce a list. Later we'll define two 
more, `foldl` and `foldr`.

The definition of reduce shows another way of indenting `if`s. 
Indentation isn't significant in Bel and only matters insofar as 
it helps humans read your code, but I've found three ways of 
indenting ifs that work well. If an if has more than two tests and 
the arguments are sufficiently short, it works well to say

```lisp
(if test1 then1
    test2 then2
          else)
```

We saw this in `all` and `some`. But if you only have one test, or some 
arguments are too long to fit two on one line, then it works better 
to say

```lisp
(if test1
    then1
    test2
    then2
    else)
```

or if an `if` is long, 

```lisp
(if test1
     then1
    test2
     then2
    test3
     then3
     else)
```

The next function, `cons`, has the name that `join` had in McCarthy's
Lisp. It's the function you use to put things on the front of a list.

```lisp
> (cons 'a '(b c))
(a b c)
```

If you only want to put one thing on the front of a list, you could
use `join`.

```lisp
> (join 'a '(b c))
(a b c)
```

With `cons`, however, you can supply more than one thing to put on the
front:

```lisp
> (cons 'a 'b 'c '(d e f))
(a b c d e f)
```

Since `cons` is a generalization of `join`, it's rare to see `join` in
programs.

We see something new in the definition of `cons`: it has a single 
parameter, `args`, instead of a list of parameters. When a function has 
a single parameter, its value will be a list of all the arguments 
supplied when the function is called. So if we call cons thus

```lisp
(cons 'a 'b '(c d))
```

the value of `args` will be

```lisp
(a b (c d))
```

The parameter list of a Bel function can be a tree of any shape. If 
the arguments in the call match its shape, the parameters will get 
the corresponding values; otherwise an error is signalled.

So for example if a function `f` has the parameter list

```lisp
(x . y) 
```

and it's called

```lisp
(f 'a 'b 'c)
```

then `x` will be `a`, and `y` will be `(b c)`.

If the same function is called

```lisp
(f 'a)
```

then `x` will be `a`, and `y` will be nil. And if it's called

```lisp
(f)
```

you'll get an error because there is no value for `x`.

If a function `f` has the parameter list 

```lisp
((x y) z)
```

and is called

```lisp
(f '(a (b c)) '(d))
```

then `x` will be `a`, `y` will be `(b c)`, and `z` will be `(d)`. Whereas if it's 
called

```lisp
(f '(a) '(d))
```

you'll get an error because there is no value for `y`, and if it's 
called

```lisp
(f '(a b c) '(d))
```

you'll get an error because there is no parameter for `c`.

The next function, `append`, joins lists together:

```lisp
> (append '(a b c) '(d e f))
(a b c d e f)
> (append '(a) nil '(b c) '(d e f))
(a b c d e f)
```

Its definition will be easier to understand if we look first at a 
two-argument version.

```lisp
(def append2 (xs ys)
  (if (no xs)
      ys
      (cons (car xs) (append2 (cdr xs) ys))))
```

In English, if `xs` is empty, then just return `ys`. Otherwise return the
result of consing the first element of `xs` onto append of the rest of 
`xs` and `ys`. I.e. 

```lisp
(append2 '(a b c) '(d e f))
```

becomes

```lisp
(cons 'a (append2 '(b c) '(d e f)))
```

and so on. The definition of `append` in the Bel source is the same 
principle applied to any number of arguments.

In it we see the first use of `apply`. Like `if`, `apply` is a special
form, meaning an operator whose behavior has to be defined as a
special case in the interpreter. Unlike `if`, apply has a value; it
evaluates to itself, like `t` and `nil`. This lets you use it as an
argument like an ordinary function. 

Its purpose is in effect to spread out the elements of a list as if 
they were the arguments in a function call. For example

```lisp
(apply f '(a b))
```

is equivalent to 

```lisp
(f 'a 'b)
```

In the general case apply can take one or more arguments, and is 
equivalent to calling `apply` on the first argument and all the 
intervening arguments `cons`ed onto the last. I.e.

```lisp
(apply f x y z)
```

is equivalent to

```lisp
(apply f (cons x y z))
```

It's common to use `apply` in functions like append that take any 
number of arguments. Using apply is in a sense the converse of using 
a single parameter to collect multiple arguments.

Now let's look at `append`. It takes any number of arguments. 
Collectively (i.e. as a list) they'll be the value of `args`. If `args`
is empty or only has one element, then the result is `(car args)`. We 
saw the same sort of test in the first clause of `reduce`. That's two
base cases, and there is also a third: when `args` has more than one
element but the first element is `nil`. In that case we can ignore it,
and apply `append` to the rest of `args`.

Finally in the last clause we see the general case. It uses the 
same strategy we saw in `append2`: `cons` the first element of the
first argument onto a recursive call to append on the rest of the 
first argument and the remaining arguments. Unlike `append2`, `append`
has to make this call using `apply`, because it has a varying number of
arguments in a list, instead of exactly two.

Once we have `append` it's easy to define `snoc`, which as its name
suggests is like a reverse `cons`,

```lisp
> (snoc '(a b c) 'd 'e)
(a b c d e)
```

and `list`, which returns a list of its arguments.

```lisp
> (list)
nil
> (list 'a)
(a)
> (list 'a 'b) 
(a b)
```

Or more precisely, returns a newly made list of its arguments. If
you're wondering why we bother appending `args` to `nil` rather than 
simply returning it, the reason is that appending a list to `nil` will 
also copy it.

If we defined list as

```lisp
(def list args args)
```

and it was called thus

```lisp
(apply list x)
```

then the value that `list` returned would be the same list as `x` -- not 
merely a list with the same elements, but the same pair -- meaning if
we modified the value we got from `list`, we'd also be modifying the 
object up in the calling code.

After `list` we see map, which in the simplest case returns a list of
calling its first argument on each element of its second.

```lisp
> (map car '((a b) (c d) (e f)))
(a c e)
```

However, `map` can take any number of lists, and calls its first 
argument on successive sets of elements from the others.

```lisp
> (map cons '(a b c) '(1 2 3))
((a . 1) (b . 2) (c . 3))
```

It stops as soon as one list runs out

```lisp
> (map cons '(a b c) '(1 2))
((a . 1) (b . 2))
```

Like `append`, `map` is easier to understand if we start with a version 
that takes exactly two arguments.

```lisp
(def map2 (f xs)
  (if (no xs)
      nil
      (cons (f (car xs))
            (map2 f (cdr xs)))))
```

If there are no `xs` left, then return `nil`, otherwise `cons f` of the
first element onto `map2` of `f` and the remaining elements. Pretty 
simple.

All the additional complexity of `map` comes from the need to take 
multiple lists. The parameter list becomes `(f . ls)` so that all the 
lists can be collected in `ls`. We need an additional base case in case 
there are zero of them. Checking for the end of the list, which in 
`map2` was

```lisp
(no xs)
```

now becomes 

```lisp
(some no ls)
```

because we stop as soon as any list runs out.

Then we have yet another base case, the one in which we have just one
list. That's what `map2` does, and not surprisingly, the code is the 
same as in `map2` except that `xs` becomes `(car ls)`.

Finally in the general case we call `f` on all the first elements
(which we collect using `map`) and `cons` that onto `map` of `f` on all the 
rests of the lists.

Notice that map calls itself recursively in two ways: there is the
usual "do this to the rest of the list" recursive call in the last 
line. But in the preceding line we also use `(map car ls)` to collect 
the arguments for `f`. And that's why we need the single-list base 
case. Without it, we'd get an infinite recursion.

Next comes our first macro, fn. There are two concepts to explain 
first, though: macros and scope. 

A macro is essentially a function that generates code. I would have 
liked the first example of a macro to be something simpler, but `fn`
is the one we need first. So I'll introduce macros using a simpler 
macro that isn't part of Bel, then explain `fn`.

Here is a very simple macro:

```lisp
(mac nilwith (x)
  (list 'cons nil x))
```

This definition says that whenever you see an expression like

```lisp
(nilwith 'a)
```

transform it into 

```lisp
(cons nil 'a)
```

and then evaluate that and return its value as the value of the call 
to nilwith.

```lisp
> (nilwith 'a)
(nil . a)
```

So unlike the evaluation of a function call, the evaluation of a 
macro call has two steps: 

1. First use the definition of the macro to generate an expression, 
   called the macro's expansion. In the case above the expansion is
   `(cons nil 'a)`.

2. Then evaluate the expansion and return the result. The expansion 
   above evaluates to `(nil . a)`.

Beneath the surface, what's going on is quite simple. A macro is in 
effect (or more precisely, contains) a function that generates 
expressions. This function is called on the (unevaluated) arguments 
in the macro call, and whatever it returns is the macro expansion.

For example, the value of `nilwith` will be equivalent to

```lisp
(lit mac (lit clo nil (x) (list 'cons nil x))))
```

If we look at the third element, we see the function that generates 
expansions

```lisp
(lit clo nil (x) (list 'cons nil x))
```

which looks just like the definition of `nilwith`. 

Macros often use backquote to make the code that generates
expressions look as much as possible like the resulting expressions.
So if you defined nilwith you'd probably do it not as we did above
but as

```lisp
(mac nilwith (x)
  `(cons nil ,x))
```

Now let's work our way up to fn, starting with the following 
simplified version:

```lisp
(mac fn- (parms expr)
  `(lit clo nil ,parms ,expr))
```

This is less powerful than the actual fn macro in two ways. 

1. It doesn't capture the local lexical environment, but instead
   simply inserts a nil environment.

2. It can only take a single expression.

But if we don't need either of these things, the functions made by
`fn-` work fine:

```lisp
> ((fn- (x y) (cons x y)) 'a 'b)
(a . b)
```

All the extra complexity in the definition of `fn` is to get those two 
features, the local environment and a body of more than one 
expression.

A function with a lexical environment stored within it is called a 
closure. That's why literal Bel functions begin `(lit clo ...)`; the
`clo` is for "closure." If a closure includes an environment with a 
value for `x`, then `x` will have a value within the closure even if it 
isn't a parameter.

So far the literal functions we've seen have had `nil` enviroments.
Let's try making one by hand with some variable bindings in it. An 
environment is a list of `(var . val)` pairs, so to make a closure we 
put such a list in the third position of a `clo`, like this:

```lisp
(lit clo ((x . a)) (y) (cons x y))
```

This closure includes an environment in which `x` has the value `a`. It 
has one parameter, `y`, and when called returns the value of `x` consed 
onto whatever we give as an argument.

```lisp
> ((lit clo ((x . a)) (y) (cons x y)) 'b)
(a . b)
```

Notice that the `b` was all we passed to the function. The a came from
within it.

It turns out to be very useful if functions include the lexical 
environment where they're created. And that is what the `fn` macro 
does.

In Bel you can get hold of the global and lexical environments using 
the variables `globe` and `scope` respectively. So for example if we 
define `foo` as

```lisp
(def foo (x)
  scope)
```

then it will work something like this

```lisp
> (foo 'a)
((x . a))
> (foo 'b)
((x . b))
```

I say "something like" because a repl may have some variables of its 
own, but we know that `scope` will at least have a value for `x`.

If you compare the definitions of `fn-` and `fn`, you'll notice that
while `fn-` expands into a `lit` expression, `fn` expands into a call to
`list` that yields a `lit` expression. It works fine to use a call to
`list` rather than an actual list in a function call; functions are 
just lists after all.

```lisp
> ((list 'lit 'clo nil '(x) '(+ x 1)) 2)
3
```

The reason the definition of `fn` expands into a call to `list` is so 
that we can incorporate the local environment, which we get by 
including `scope` in the arguments to `list`.

Here's an example where we do this manually:

```lisp
(def bar (x)
  ((list 'lit 'clo scope '(y) '(+ x y)) 2))
```

Within `bar` we call a hand-made closure that includes `scope`, which, as
we know from the example of foo above, will include a value for `x`.

```lisp
> (bar 3)
5
```

The fn macro generates as its expansion exactly what we just made by 
hand. So this is equivalent to the definition above:

```lisp
(def bar (x)
  ((fn (y) (+ x y)) 2))
```

The `fn` macro has two different expansions depending on how many 
arguments we pass to it. That's so that functions can have bodies of
more than one expression.

If we call `fn` with two arguments, meaning a parameter list and an
expression, as in e.g.

```lisp
(fn (x) (cons 'a x))
```

then `(cdr body)` will be false, so the expansion will be

```lisp
(list 'lit 'clo scope '(x) '(cons 'a x))
```

If we call `fn` with three or more arguments, meaning a parameter list 
plus two or more expressions, e.g.

```lisp
(fn (x) 
  (prn 'hello) 
  (cons 'a x))
```

Then the expansion wraps a `do` around the expressions.

```lisp
(list 'lit 'clo scope '(x) '(do (prn 'hello) (cons 'a x)))
```

We haven't seen `do` yet, but it's coming soon. It makes multiple
expressions into a block of code. 

Next comes something unusual: vmark is set to a newly created pair 
made by join. Missing arguments to primitives default to `nil`, so 
`(join)` is equivalent to `(join nil nil)`, and when you see a call like 
this, it's usually for the purpose of creating a fresh pair to mark 
the identity of something.

Any pair with `vmark` in its `car` is treated by Bel as a variable. The 
next function, `uvar`, thus returns a new, unique variable each time 
it's called. The reason we need such a thing is so that when we're
manipulating user code, we can add variables without worrying they'll 
accidentally share the names of variables created by users.

Now we see the definition of `do`, which we used in the expansion of 
`fn`. The `do` macro uses nested function calls to represent blocks of 
code.

Suppose you want to evaluate two expressions in order and then return
the value of the last.

```lisp
e1
e2
```

You can make this happen by embodying `e2` in a function that you then 
call on `e1`. 

```lisp
((fn x e2) e1)
```

When this expression is evaluated, e1 will be evaluated first, and 
its value passed to the function in the car of the call:

```lisp
(fn x e2)
```

Then `e2` will be evaluated, ignoring the value of e1 passed in the 
parameter, and its value returned. Result: `e1` and then `e2` get 
evaluated, as if in a block. (You cannot of course safely use `x` as 
the parameter in case it occurs within `e2`, but I'll explain how to 
deal with that in a minute.) 

This technique generalizes to blocks of any size. Here's one with 3 
expressions:

```lisp
((fn x ((fn x e3) e2)) e1)
```

You can use `reduce` to generate this kind of expression as follows

```lisp
(def block args
  (reduce (fn (x y)
            (list (list 'fn 'x y) x))
          args))

> (block 'e1 'e2 'e3)
((fn x ((fn x e3) e2)) e1)
```

and this is almost exactly what the `do` macro does. If you look at
its definition, it's almost identical to that of `block`. 

One difference is that `do` is a macro rather than a function, which
means that the nested call gets evaluated after it's generated.

The other difference is that we call `uvar` to make the parameter 
instead of using `x`. We can't safely use any symbol as the parameter
in case it occurs in one of the expressions in the do. Since we're 
never going to look at the values passed in these function calls, we 
don't care what parameter we use, so long as it's unique.

If you want to establish a lexical binding for some variable, you do
it with `let`, which is a very simple macro upon `fn`.

```lisp
> (let x 'a 
    (cons x 'b))
(a . b)
```

Since `let` expands into a `fn`, you have the full power of Bel parameter 
lists in the first argument.

```lisp
> (let (x . y) '(a b c) 
    (list x y))
(a (b c))
```

The `macro` macro is analogous to the `fn` macro in that it returns a
literal macro. You'll rarely use these directly, but you could if you 
wanted to.

```lisp
> ((macro (v) `(set ,v 'a)) x)
a
> x
a
```

Next we see the definition of `def` itself, which does nothing more
than set its first argument to a `fn` made using the rest of the
arguments, and also of `mac`, which does the same with macro.

(I like it when I can define new operators as thin, almost trivial
seeming layers on top of existing operators. It seems a sign of
orthogonality.)

If you were wondering why `fn` needs two cases -- why we don't just 
always wrap a do around the body -- the reason is that `do` calls 
`reduce`, which is defined using `def`, which expands into a `fn`. So to 
avoid an infinite recursion we either have to define `reduce` as a 
literal function, or make either `fn` or `do` consider the single 
expression case, and making `fn` do it was the least ugly.

Now that we have `let`, we can define `or`, which returns the first 
non-nil value returned by one of its arguments. Like most ors in
programming languages, it only evaluates as many arguments as it
needs to, which means you can use it for control flow as well as
logical disjunction.

```lisp
> (or 'a (prn 'hello))
a
```

The definition of `or` is the first recursive macro definition we've
seen. Unless it has no arguments, `an` or will expand into another `or`.  
This is fine so long as the recursion terminates, which this one 
will because each time we look at the `cdr` of the list of arguments, 
which will eventually be `nil`. (Though you could spoof or by 
constructing a circular list and applying or to it.)

In effect

```lisp
(or foo bar)
```

expands into

```lisp
(let x foo
  (if x 
      x
      (let y bar
        (if y
            y
            nil))))
```

except that we can't actually use variables like `x` and `y` to hold the 
values, and instead have to use `uvar`s.

Notice incidentally that the expression above could be optimized

```lisp
(let x foo
  (if x
      x
      bar))
```

but the definition of `or` doesn't try to; like every definition in
Bel, its purpose is to define what or means, not to provide an 
efficient implementation of it.

In Bel, macros are `apply`able just like functions are, though if you
do that you get only the logical aspect of `or` and not the control
aspect, since in a call to apply the arguments have already all been
evaluated.

```lisp
> (apply or '(nil nil))
nil
> (apply or '(nil a b))
a
```

The `and` macro is similar in spirit to `or`, but different in its 
implementation. Whereas or uses recursion to generate its expansion, 
and uses reduce. Since 

```lisp
(and w x y z) 
```

is equivalent to

```lisp
(if w (if x (if y z)))
```

it's an obvious candidate for reduce. 

Notice the function given to `reduce` has a single parameter. A 
function given as the first argument to reduce will only ever be 
called with two arguments, so usually such a function will have a 
list of two parameters, but in this case we just want to `cons` an `if` 
onto the front of the arguments each time.

The other interesting thing about and is what we do when it has no 
arguments. While we want `(or)` to return nil, we want `(and)` to return 
t. So in the second argument to reduce, we replace an empty args with 
`(t)`.

The next function, `=`, is the one that programs usually use to test 
for equality. It returns true iff its arguments are trees of the same 
shape whose leaves are the same atoms.

```lisp
> (id '(a b) '(a b))
nil
> (= '(a b) '(a b))
t
```

In Bel, everything that's not a symbol, character, or stream is a
pair. Numbers and strings are pairs, for example. So you'd never
want to use id for comparison unless you were specifically looking
for identical list structure.

In the definition of `=` we see the first instance of square bracket
notation. 

```lisp
(all [id _ (car args)] (cdr args))
```

This is equivalent to

```lisp
(all (fn (x) (id x (car args))) 
     (cdr args))
```

I.e., is everything in the `cdr` of args `id` to the `car`? You know you
can use `id` to test equality at this point, because if one of the args 
is an atom, they all have to be for them to be `=,` and you can use `id` 
to test equality of atoms.

If `id` took any number of arguments (it doesn't, because I want axioms
to be as weak as possible), the preceding all expression could have 
been simply

```lisp
(apply id args)
```

The next four functions are predicates for the four types. All use `=` 
for this test even though all could use `id`. My rule is to use `=`
unless I specifically need `id`. That way the appearance of `id` is a 
signal that code is looking for identical structure.

Then we see `proper`, which tells us whether something is a proper
list. Informally, a proper list is one that we don't need a dot to
display.

```lisp
> (proper nil)
t
> (proper '(a . b))
nil
> (proper '(a b))
t
```

Formally, something is a proper list if it's either `nil` or a pair
whose `cdr` is a proper list, which is exactly what the definition
says.

In Bel, a proper list of characters is called a string, and has a 
special notation: zero or more characters within double quotes.

```lisp
> (string "foo")
t
```

The next function, mem, tests for list membership.

```lisp
> (mem 'b '(a b c))
(b c)
> (mem 'e '(a b c))
nil
> (mem \a "foobar")
"ar"
```

Since it uses `some`, it returns the rest of the list starting with the 
thing we're looking for, rather than simply `t`.

In the definition of mem we see the first use of an optional 
parameter. If in a parameter list you see a list beginning with the 
symbol `o`, then the parameter following the `o` is an optional one. It 
can be followed by an expression, and if no value is supplied for the
parameter in the call, it gets the value of that expression instead 
(or `nil` if there isn't one). So

```lisp
> ((fn (x (o y)) y) 'a)
nil
> ((fn (x (o y 'b)) y) 'a)
b
```

The optional parameter doesn't have to be a single symbol; it can be 
a full parameter list. An example will be easier to understand if we 
use let:

```lisp
> (let (x (o (y . z) '(a . b))) '(f)
    (list x y z))
(f a b)
```

In the definition of `mem`, the optional parameter is a comparison
function, which defaults, as such functions usually will, to `=`.
By supplying different comparison functions we can get different 
behaviors out of `mem`.

```lisp
> (mem 3 '(2 4 6 8) >)
(4 6 8)
```

The next function, `in`, is effectively a generalization of `=`. It 
returns true iff its first argument is `=` to any of the rest.

Then come three common combinations of `car` and `cdr`: `cadr`, which gets 
the second element of a list, `cddr`, which takes two elements off the 
front, and `caddr`, which gets the third element. We'll have other ways 
to do these things once we've defined numbers.

The `case` macro takes an initial expression `e`, followed by 
alternating keys (which are implicitly quoted) and expressions, and 
returns the result of evaluating the expression following the key 
that's `=` to the value of `e`.  

If `case` is given an even number of arguments, the last one is the 
default. Otherwise the default is `nil`.

E.g. this function 

```lisp
(def sname (s)
  (case s 
    + 'plus
    - 'minus
      'unknown))
```

returns the name of a sign represented by a symbol:

```lisp
> (sname '+)
plus
```

The `iflet` macro lets you use the result of a test in an `if`. It works 
like an ordinary `if`, except that it takes an initial variable, which
in any then expression will be lexically bound to the value returned 
by the preceding test expression.

```lisp
> (iflet x nil      'foo
           '(a b c) (car x)
                    'bar)
a
```

Notice how similar the definitions of `case` and `iflet` are, despite 
their different purposes. They're both recursive macros, like `or`, 
and both work through their arguments two at a time.

We use `iflet` to define `aif`, which implicitly binds the variable `it`
to the value of the preceding test expression.

```lisp
> (map (fn (x)
         (aif (cdr x) (car it)))
       '((a) (b c) (d e f) (g)))
(nil c e nil)
```

The function given to `map` here tests whether x has a non-nil cdr, 
and if so returns the car of `it`.

With `aif` and `some`, it's trivial to define `find`, which returns the
first element of a list that matches some test.

```lisp
> (find [= (car _) \a] 
        '("pear" "apple" "grape"))
"apple"
```

The `begins` function returns true iff its first argument is a list 
that begins with its second argument:

```lisp
> (begins '(a b c d e) '(a b))
t
```

Like `mem`, it takes an optional comparison function that defaults 
to `=`.

It's used in `caris`, which returns true iff its first argument is a 
pair whose car is its second.

```lisp
> (caris '(a b) 'a)
t
```

This is one of those functions you end up using surprisingly often, 
because it's so common for the car of a list to have some special 
significance.

Our next function, `hug`, applies a function to pairs of elements of a 
list. Since the default function is list, by default it simply 
returns pairs of elements.

```lisp
> (hug '(a b c d e))
((a b) (c d) (e))
> (hug '(1 2 3 4 5) +)
(3 7 5)
```

This too is something you need surprisingly often, especially when 
operating on expressions, where it's common to have subexpressions 
that form implicit pairs. We've seen this already in `if`, `case`, and 
`iflet`, and we see it implemented with `hug` in the next macro, `with`, 
which is a multi-variable `let`.

```lisp
> (with (x 'a 
         y 'b) 
    (cons x y))
(a . b)
```

It binds the variables in parallel in the sense that the bindings of 
previous variables are not visible in the expressions defining the 
values of later ones.

```lisp
> (let x 'a
    (with (x 'b
           y x)
      y))
a
```

The next function, `keep`, returns all the elements of a list that pass 
some test

```lisp
> (keep odd '(1 2 3 4 5))
(1 3 5)
```

and `rem` removes its first argument from a list

```lisp
> (rem \a "abracadabra")
"brcdbr"
```

or more precisely, since it takes an optional comparison function `f`, 
all the elements of the list that fail `[f _ x]`, where `x` is the first 
argument.

```lisp
> (rem 4 '(5 3 1 2 4) >=)
(3 1 2)
```

The next two functions, `get` and `put`, are for operating on key-value
stores represented as lists of pairs like this one:

```lisp
> (set x '((a . 1) (b . 2) (c . 3)))
((a . 1) (b . 2) (c . 3))
```

The first, `get`, retrieves entries,

```lisp
> (get 'a x)
(a . 1)
> (get 'z x)
nil
```

and the second, `put`, creates them.

```lisp
> (put 'z 26 x)
((z . 26) (a . 1) (b . 2) (c . 3))
> (put 'a 9 x)
((a . 9) (b . 2) (c . 3))
```

The reason `get` returns the whole pair instead of just the associated
value is so that we can distinguish between a key having a value of 
`nil` and having no value.

Notice that `put` doesn't change the value of `x`, just as `cons`ing 
something onto `x` wouldn't change the value of it. 

The function `rev` reverses a list,

```lisp
> (rev "able")
"elba"
```

and `snap` breaks off a piece of its second argument that's as long as
its first, returning both parts:

```lisp
> (snap '(a b) '(1 2 3 4 5))
((1 2) (3 4 5))
```

It's used in `udrop` (for "unary drop"), which returns just the 
remaining part:

```lisp
> (udrop '(a b) '(1 2 3 4 5))
(3 4 5)
```

Then we get the identity function, `idfn`:

```lisp
> (map idfn '(a b c))
(a b c)
```

You wouldn't call this directly (why bother?) but you often end up
using it as a default or when operating on functions.

The function `is` is a little unusual in that it returns a function for
comparing its argument to something. 

```lisp
> ((is 'a) 'a)
t
```

An `is` is a partially applied `=`, so in principle we won't need it 
after we define partial application later on. But this case is so
common that it's convenient to have a separate operator for it.

Now come several macros for dealing with errors. The first, `eif`,
introduces several new concepts, so I'll explain them first, then `eif` 
itself.

One thing we see being used for the first time here is dynamic 
binding. To show how it works, we'll define a function that refers to 
a variable with no global value:

```lisp
(def foo ()
  snerg)
```

If we call foo normally, we'll get an error saying `snerg` has no 
value. But if we call `foo` within a `dyn` expression that creates a
dynamic binding for `snerg`, it will work:

```lisp
> (dyn snerg 'a 
    (foo))
a
```

We couldn't get the same result by saying

```lisp
(let snerg 'a
  (foo))
```

because a lexical binding created by `let` (or more precisely by a
function call) is only visible within its body. And whereas lexical 
bindings get saved in closures (as in e.g. `is`), dynamic bindings, 
like global ones, don't.

Another concept we're seeing for the first time is that of a 
continuation. A continuation is basically a computation in the middle 
of happening. (Or more prosaically, it's a copy of the stack.) A 
continuation is callable, like a function, and if you call one, you 
restart the computation where it was created.

You can get your hands on the current continuation by calling the `ccc` 
special form with a function of one argument. It will be the current 
continuation, which you can then save. Let's try making one.

Here's some simple code that makes a list:

```lisp
> (list 'a 'b)
(a b)
```

Now let's try replacing the 'b with an expression that saves the 
current continuation before returning b:

```lisp
> (list 'a (ccc (fn (c) 
                  (set cont c) 
                  'b)))
(a b)
```

It returns the same value, but in the process we've set cont to the
continuation at the point where the `ccc` occurred. If we call `cont` 
with some value, our old computation will be restarted as if that 
value had been returned by the `ccc` expression:

```lisp
> (cont 'z)
(a z)
```

Continuations work any number of times:

```lisp
> (cont 'w)
(a w)
```

One thing we can use continuations for is aborting computations. If 
we save a continuation before starting something, then by calling the 
continuation we can escape from the middle of it.

```lisp
> (ccc (fn (c) 
         (dyn abort c 
           (do (abort 'a)
               (car 'b)))))
a
```

Here we bind `abort` to the continuation before we start evaluating the 
do expression. The second expression within the `do`

```lisp
(car 'b)
```

would cause an error if it were evaluated. But we never get to it, 
because we call `abort` first.

When an error occurs, `err` is called on a value representing the
error. So if the variable that we dynamically bind is `err` instead of
`abort`, we can take over what happens when an error is signalled.

Here we rebind err to return hello when an error occurs:

```lisp
> (ccc (fn (c) 
         (dyn err (fn (x) (c 'hello))
           (car 'b))))
hello
```

This time the car expression does get evaluated, which causes an 
error to be signalled. But by establishing a dynamic binding for
err, we've ensured that it's our function that gets called when the
error is signalled. And our function simply returns `hello` from the 
`ccc` expression. 

You can probably imagine how you'd write a macro to evaluate an 
expression in an error-proof way: just make the expansion put the
expression within something like

```lisp
(ccc (fn (c)
       (dyn err (fn (x) (c nil))
         expression)))
```

except of course you'd want to use a `uvar` instead of `c`.

Now let's look at `eif`. It's like if except that which of its 
arguments get evaluated depends not on whether its test expression
returns true, but whether it causes an error.

```lisp
> (eif x (car 'a)
         'oops
         x)
oops
> (eif x (car '(a b))
         'oops
         x)
a
```

The variable before the test expression (in this case `x`) will be 
lexically bound either to the value returned by the test expression, 
or to whatever err was called with if an error occurred.

The expansion of the first `eif` above looks like

```lisp
(let v (join)
  (let w (ccc (fn (c)
                (dyn err [c (cons v _)] (car 'a))))
    (if (caris w v id)
        (let x (cdr w) 'oops)
        (let x w x))))
```

except of course `v`, `w`, and `c` will be uvars. When we look at the code
above, we can see how `eif` tells whether the value it got back from 
the test expression represents an error or not. The variable `v` is 
bound to a newly created pair. Within the continuation, `err` is bound 
to a function that returns `v` consed onto whatever `err` was called 
with. So to decide which of the two succeeding expressions to
evaluate, we just check whether the `car` of `w` is `v`. (And of course we 
check using `id`, not `=`.)

The `eif` macro is the most general error-catching macro, but there
are two more, `onerr` and `safe`, that are more commonly used. The
`onerr` macro takes two expressions and returns the value of the first
if the second causes an error:

> (onerr 'oops (car 'a))
oops

and the `safe` macro simply returns `nil` if the expression within
it causes an error:

```lisp
> (safe (car '(a b)))
a
> (safe (car 'a))
nil
```

The next function, `literal`, returns true iff its argument evaluates 
to itself,

```lisp
> (map literal (list nil "foo" car))
(t t t)
```

while `variable` returns true iff its argument is a variable, meaning
an ordinary symbol or a `uvar`:

```lisp
> (map variable (list 'x (uvar) t))
(t t nil)
```

And `isa` is for checking whether something is a particular kind of 
`lit`. Like `is`, `isa` doesn't do the check, but returns a function that
does

```lisp
> ((isa 'clo) map)
t
```

The operators we've defined so far, together with the axioms, will 
now enable us to define a function that acts as a Bel interpreter: a 
function that will take any Bel expression as an argument and 
evaluate it.

(We don't need all the operators we've defined so far to define a Bel 
interpreter. These are, rather, the minimum set we need to define an 
interpreter in a way that's not too ugly.)

Much of the code in the interpreter operates on the same set of
structures, and these always have the same parameter names.

Each thread is a list 

```lisp
(s r)
```

of two stacks: a stack `s` of expressions to be evaluated, and a stack 
`r` of return values.

Each element of `s` is in turn a list

```lisp
(e a)
```

where `e` is an expression to be evaluated, and `a` is a lexical
environment consisting of a list of `(var . val)` pairs.

The variable `p` holds a list of all the threads (usually other than 
the current one).

The other thing we need to pass around in the interpreter is the
global bindings, which is another environment represented as a list 
of `(var . val)` pairs. I use the variable `g` for this.

The most common parameter list we'll see is 

```lisp
(s r m)
```

where `s` is the current expression stack, `r` is the current return 
value stack, and `m` is a list `(p g)` of the other threads and the 
global bindings.

The interpreter itself begins with the function `bel`, which takes an 
expression `e` and starts the interpreter running with a single thread 
in which `e` is to be evaluated. The arguments it sends to ev represent 
the usual `(s r m)` triple. So 

```lisp
(list (list e nil))
```

is an expression stack containing nothing except `e`, to be evaluated 
in a null environment. The second argument, `nil`, is the return value 
stack, which is empty because we're not returning from anything. And 
the third argument is `m`, aka `(p g)`, a list of the other threads 
`(currently nil)` and an environment to use as the global bindings.

If we jump ahead a few definitions to `ev`, we come to the core of the 
interpreter. This function plays the role `eval` did in McCarthy's 
Lisp. Its parameters implicitly pull an `(e a)` expression-environment 
pair off the expression stack. There are only five things the 
expression can be:

1. A literal, in which case we return it.

2. A variable, in which case we call `vref` to look up its value.

3. An improper list, in which case we signal an error.

4. A list beginning with a special form, in which case we call the 
   associated function stored in forms.

5. An ordinary call, in which case we call `evcall` on it.

I'm going to follow the trail of evaluating a literal to explain some 
things about how evaluation works, then come back and examine the 
other cases.

One of the most important things to understand about the 
interpreter is that it never returns a value till it terminates. 
The way it implements returning a value in the program it's
evaluating is not by returning a value itself, but by a recursive 
call to the interpreter with a shorter expression stack and the 
return value `cons`ed onto the return stack. And that's what we see 
happening in the code that runs when `e` is a literal:

```lisp
(mev s (cons e r) m)
```

That is what returning a value looks like.

The function `mev` (`m` = multi-threaded) is what the interpreter calls 
to continue evaluation after doing something. Its purpose is to check 
whether interpretation should terminate, and if not, to allow another 
thread to run.

The first thing `mev` does is check if the current thread has run out 
of work to do. If so, if `s` is `nil`, it checks whether there are other 
threads in `p`. If there are, it calls `sched` to run one. If not, if 
this is the only thread and we've just finished it, then it returns 
whatever's on top of the return value stack as the value of calling 
the interpreter.

If we haven't finished the current expression stack, then we have to 
check whether we should stay in this thread or switch to another one. 
Ordinarily you want to give other threads a chance to run, but 
sometimes you can't, if you're doing something that requires multiple 
steps to complete, and in the middle is in an inconsistent state. 

The way a program signals that it doesn't want to be interrupted is 
by dynamically binding `lock` to a non-nil value. If `lock` is on, we put 
the current thread on the front of the list of threads, and if not we 
put it on the end. Since `sched` always runs the first thread on the 
list, if we keep the current thread on the front, it keeps running.

Now that we've seen how `mev` and `sched` work, let's return to `ev`. If `e` 
is a variable, we call `vref` to evaluate it. And what `vref` ordinarily 
does is this:

```lisp
(aif (lookup v a s g)
     (mev s (cons (cdr it) r) m)
     (sigerr 'unbound s r m))
```

You may now recognize that kind of call to `mev`: that's returning a 
value. If the lookup succeeds, it returns the `(var . val)` pair it 
found, so the value is the `cdr` of it. If lookup fails, it returns 
`nil`, in which case we've just encountered a reference to an unbound 
variable, and we should signal an error.

Let's skip down to `lookup` and see what it does. It checks, in order, 
whether the variable has a dynamic binding, a lexical binding, or a 
global binding. At the end there are special cases for the two 
variables `globe` and `scope`; for them the interpreter simply "leaks" 
the corresponding parameter. Leak is an apt metaphor in this 
situation because data is going from one layer of Bel to another: 
from the interpreter running in Bel to the Bel program it's 
evaluating.

We use binding to check whether a variable has a dynamic binding. It
checks by searching the expression stack looking for an entry binding 
that variable. As we'll see when we get to its definition, `dyn` works 
by inserting a special entry on the expression stack listing the 
variable it wants to bind and its value. There are other operators 
that insert special entries on the expression stack too. These 
entries are distinguishable from ordinary expressions by beginning 
with a pair called `smark`.

While we're here, let's look at `sigerr`. This is how the interpreter
signals an error. As we saw earlier in the examples of continuations,
it begins by looking for a dynamic binding for `err`, and if there is
one, it calls it using `applyf`, which we'll get to later. If there
isn't a binding for `err`, then there's an error in the interpreter 
itself, and we call `err` about it. 

This sort of code where things happen at two different levels --
the Bel instance running the interpreter, and the Bel program that
the interpreter is evaluating -- is inevitably a bit confusing, but 
that comes with the territory when a language is written in itself. 
For errors, at least, there is a simple rule of thumb: when there's 
an error in a program you're evaluating, you call `sigerr`, and when 
there's an error in yourself, you call `err`.

Calling `sigerr` is like a parent reporting that the baby is crying.
Calling `err` is crying oneself.

But what about the first half of `vref`, the one in which `(inwhere s)`
returns true? This code is for handling assignments. In Bel, any
value that lives in a pair can be assigned using `set`. For example,

```lisp
> (let x '(a b c)
    (set (cadr x) 'z)
    x)
(a z c)
```

This code in `vref` is what makes it happen. It works as follows. When 
we're in a `set`, another special entry (beginning as usual with `smark`) 
is put on the stack saying that we're not looking for the value of 
something, but its location. The function `inwhere` checks for one of 
these, and if it finds one, we return not the value we're looking up, 
but the pair it occurs in, plus either `a` or `d` to say whether it's in 
the `car` or the `cdr`.

New global variables are created implicitly by assigning values to 
them, and this is where that happens. The special stack entry found 
by inwhere says whether a new global binding should be created (it 
should be for `set`, but not for `push` for example) and if it should, we 
create a new pair to hold its name and value, and splice it into the 
global variables.

Notice that in this case the first argument to `mev`, at the end, is 
`(cdr s)` rather than `s`. We're discarding the special stack entry found 
by inwhere.

Incidentally, this is a case where we need `lock`, because when this 
new binding is created, the value is `nil` initially. So if code in 
another thread looked up this variable, there would be a window 
during which it seemed to have the value `nil`. The code that handles 
locking is in the definition of `set`, which we'll see later.

Now back up to `ev`. The next case is when `e` isn't a proper list. We 
know by this point that `e` not an atom, because every atom is either a 
literal or a variable. So `e` must be a list, but we have to make sure 
it's a proper one. If it isn't we call `sigerr`.

By the next line we know that `e` is a nonempty proper list. So the 
next question is, is the first element a special form?  The variable 
`forms` is a list of `(name . function)` pairs telling the interpreter 
what to do when it encounters an expression beginning with name. 

The first of these is `smark`, the pair we created for marking stack 
entries that are not just ordinary expressions. The initial value of 
`forms`

```lisp
(set forms (list (cons smark evmark)))
```

means that when the expression we've just taken off the stack 
begins with `smark`, we should call `evmark`. And in the definition of 
`evmark` we see the four types of stack entries we use `smark` for, each 
indicated by the symbol that comes after `smark` in the expression we 
put on the stack.

One of these, `bind`, we've already talked about. This is what `dyn` puts 
on the stack to establish a dynamic binding. When we return back 
through one, all that should happen is that the dynamic binding 
ceases to exist. The bind entry itself is a no-op; we just call 
`(mev s r m)` and get on with things.

We're also familiar with the idea of a `loc` entry; that's what we were 
just looking for in `inwhere`. We made a point of removing it from the 
stack once we'd found it. So if we return up through a `loc` entry, 
that means we were unable to find a pair to modify, and we should 
signal an error. E.g.

```lisp
> (set \a 5)
Error: unfindable
```

A `fut` (future) stack entry contains a closure to be called on 
`(s r m)`. The interpreter inserts these when it needs to do 
something in several steps. This happens so often that there's a 
special `fu` macro (the resemblance to `fn` is intentional) for creating 
these closures.

There's one of them in the `prot` clause of `evcall`. A `prot` (protect)
stack entry means that an expression should be evaluated even when 
calling a continuation throws control back over it. You'd use this 
for example to make sure a file you opened eventually got closed. 
Since these protected expressions aren't supposed to return values, 
the code for evaluating them includes a `fu` for throwing away the 
value that gets put onto the return value stack after the expression 
is evaluated: all this `fu` does it turn a call of the form `(mev s r m)` 
into `(mev s (cdr r) m)`. We'll learn more about protected expressions 
when we get to the code for creating them.

The rest of the special forms are defined using the form `macro`, which 
runs its arguments through formfn and then puts the result in an 
entry in forms. Let's start by looking at an example of a special 
form defined with it. The first is the definition of `quote`, which is 
about as simple as a special form can get. The first parameter, `(e)`, 
is the parameter list of the form as it will be called. So `(e)` means 
that quote will take exactly one argument. We call it `e`, for 
expression, because it won't have been evaluated. The remaining 
arguments, `a s r m`, represent the lexical environment plus the usual 
`s r m` state of the world.

The body of the form is a familiar type of call to `mev`, representing
a return. So this definition says that `quote` takes one argument, `e`, 
and returns it unevaluated.

The definition of `quote` will be transformed by `formfn` into the 
equivalent of

```lisp
(fn v
  (eif w (apply (fn (e) (list e))
                (car v))
         (apply sigerr 'bad-form (cddr v))
         (let (e) w
           (let (a s r m) (cdr v) 
             (mev s (cons e r) m)))))
```

The test expression in the `eif` accumulates all the form's parameters.
If an error occurs while doing that, we signal an error. (We know 
`(cddr v)` will be the familiar `(s r m)`.) Otherwise we bind the 
parameters and evaluate the body of the form.

The reason we do this in two parts is in case someone calls the form 
with the wrong arguments. If you're using form to define a form, you 
can (and must) catch all other errors yourself, but you can't catch 
that one. At least not short of using a single parameter to hold all 
the arguments, and then teasing them out manually. Since that would 
make form definitions ugly, the `form` macro does it for you.

```lisp
> (quote a b)
Error: bad-form
```

Next comes the definition of `if`. Unlike `quote`, `if` does several 
things, and when we do several things in a form, we don't just
evaluate a block of code; we put the things we need to do on the
stack and call `mev`. 

In the base case, if the `if` has no arguments, we just return `nil`,
using the familiar formula. Otherwise we want to evaluate the first 
argument and then, if there are more arguments, evaluate the 
appropriate one depending on the value of the first. Which is 
basically the code translated into English. Putting

```lisp
(list (car es) a)
```

on the stack is "evaluate the first argument," and the conditional
that puts a closure that calls `if2` on the stack, but only if `(cdr es)`
is true, is "if there are more arguments, evaluate the appropriate
one."

Notice that `if2` is just an ordinary function, not a form. It handles
the second part of evaluating an `if`, the choice of which subsequent
expression to evaluate depending on the value returned by the first.

If the value on top of the return stack is true, then the test 
expression returned true, and the whole if returns the value of the 
succeeding expression, so we just put `(car es)` onto the stack and go. 

But if the first expression returned `nil`, we need to keep going.
If we're evaluating

```lisp
(if a b c d e)
```

and `a` returns `nil`, then the value of the `if` expression is the value
of 

```lisp
(if c d e)
```

and that is exactly what `if2` is doing when it puts 

```lisp
(cons 'if (cdr es))
```

on the stack. So the `if` form is recursive in the sense that it can 
cause another `if` expression to end up on the stack, where it will of 
course be handled by the `if` form.

Regardless of what we put on the stack in `if2`, we need to discard
the value returned by the test expression, which is why the second
argument to `mev` in `if2` is `(cdr r)`.

The `if` in `if2` is where conditionalness (for lack of a better word)
gets "leaked" from the Bel in which the interpreter is running to the
Bel program that the interpreter is evaluating.

The next form, `where`, is the one that creates the stack entries that
`inwhere` looks for. It takes an expression `e` followed by an optional
argument saying whether to create a new binding for a variable if
there isn't one already. It asks for `e` to be evaluated by putting
the conventional

```lisp
(list e a)
```

on the stack. But also, adjacent to this, it puts a special `loc` entry
on the stack saying that what we want when evaluating `e` is its 
location, not its value. This will be seen by `inwhere` in functions 
like `vref` and handled accordingly.

```lisp
> (let x 'a
    (where x))
((x . a) d)
```

This return value is a list of the actual lexical binding created by
that `let`, plus a `d` indicating that the value in question is in the 
`cdr` of the pair. It's hard to imagine a reason to call where in 
ordinary code; you use it in macros like set that modify things.

Notice incidentally that the second, optional argument to `where` isn't
evaluated. It's inserted directly into the `loc` entry built by `where`.

After where, we see another form we're somewhat familiar with: `dyn`,
which establishes dynamic bindings. This definition is like that of
`if` in the sense that there's a function, `dyn2`, that does the second 
half of the work. The first half is to put the first expression, `e1` 
on the stack to get it evaluated. Its value will be what we bind the 
variable to. 

In `dyn2` we see this binding being created, in the form of a stack
entry being put on the stack. We also ask for the evaluation of the
second expression, and we put that request, `(e2 a)`, in front of the
`bind` entry. If we see a reference to `v` during the evaluation of `e2`,
the `bind` entry will be sitting on the stack to tell us it has a 
dynamic binding.

Notice that the payload of the bind entry will be a `(var . val)`
pair of the familiar form, just like we find in lexical and global
environments. This is not a coincidence. It means the code that sets 
global or lexical variables also sets dynamic bindings; in fact, the 
code doesn't even know which type it's operating on; all it knows is 
that it's doing something to the `cdr` of a pair.

```lisp
> (dyn x 'a
    (do (set x 'b) 
        x))
b
```

This wasn't a deliberate design choice. Like many other aspects of 
Bel, it's just a consequence of making the interpreter as simple as 
possible. I tried to let simplicity make as many decisions for me as 
I could.

Like `where` and `dyn`, after works by putting a special entry on the 
stack -- this time a `prot` entry. We saw what a `prot` entry means when
examining `evmark`. All the after form does is create the `prot` entry;
the real work happens elsewhere.

The `ccc` form creates a continuation, or more precisely generates
code that calls a function on a continuation. A continuation itself
is a simple thing -- basically just `s` and `r`, which between them are a 
complete representation of the state of the current thread.

Finally, the `thread` form takes an expression and starts evaluating it
in a new thread. Unusually, nearly all the work is in the last
argument to `mev`, where we build new expression and return value
stacks much as we did up in bel. The second argument `(cons nil r)` 
means the thread expression itself returns `nil` in the thread in which 
it occurs.

Along with `lit` and `apply`, those seven operators-- `quote`, `if`, `where`, 
`dyn`, `after`, `ccc`, and `thread` -- are all the special forms. But if you 
need more you can easily define them with `form`.

Now we come to the code that handles the last case in `ev`, an ordinary 
call. Its definition looks a lot like the code we saw in the forms. 
All the code in the interpreter is either calls to `mev` or to things 
that call `mev`. By now we're used to reading this sort of thing: 
`evcall` puts the car of the expression on the stack to be evaluated, 
plus a closure that calls `evcall2` to do the rest of the work.

The rest of the work, as we see in `evcall2`, begins with checking 
whether the value of the `car` of the expression is a macro. If it is, 
we call `applym`. If it isn't, we know we can evaluate the arguments, 
so we put them all on the expression stack, along with a closure that 
will snap the corresponding number of values off the top of the 
return stack, and then send them plus the operator to `applyf`.

When we look at `applym`, we see that the evaluation of a macro call 
also goes through `applyf`, but with different arguments. We apply the 
function within the macro, `(caddr mac)`, to the unevaluated arguments, 
`args`. This generates the macroexpansion. But the macroexpansion then 
has to be evaluated, and that's the purpose of this closure we put on
the stack: 

```lisp
(fu (s r m)
  (mev (cons (list (car r) a) s)
       (cdr r)
       m)
```

The `(car r)` within it is a sign of something unusual happening: we're 
moving something (the macroexpansion) from the return value stack to 
the expression stack.

There's a slight surprise waiting in `applyf`: the `apply` special form 
is implemented as a clause here. What it does is pretty simple: `apply` 
is a recursive call to `applyf` with the arguments joined together. The 
next case is the general one, when `f` is a `lit`, and in that case we 
call `applylit`.

Functions aren't the only thing you can apply in Bel. You can apply 
anything if you define a way to do it. We haven't gotten to this yet, 
but numbers for example return the corresponding element of a list:

```lisp
> (3 '(a b c d))
c
```

Such expressions are also settable:

```lisp
> (let x '(a b c d)
    (set (3 x) 'z)
    x)
(a b z d)
```

Since the thing we're about to apply might be a settable reference,
the first thing we need to check in `applyprim` is whether we're in
a where, just as we do when looking up the value of a variable. If we
are, then we look in a list called `locfns` to see if there's one that 
matches, and if so we call the corresponding function on the function 
and its arguments. 

Let's skip down to the `loc`s defined after `applyprim` to see some of
these. The `loc` operator defines new `locfns`, and after its definition
we see the `loc` for `car`. Or more precisely, a `loc` whose test will 
match the symbol `car`; the keys of `loc`s are predicates, not symbols.

We don't need many `loc`s; knowing how to set the `car` or `cdr` of a pair
will handle most cases. When we look at the `loc` for `car`, it's pretty 
simple: it just "returns" (by consing onto the return stack) a list 
of the argument and an `a`, to indicate we're looking in the `car`.

```lisp
> (let x '(a b c) 
    (where (car x)))
((a b c) a)
```

The `loc` for `cdr` is similarly simple.

After the `inwhere` check, we see cases for the four predefined 
`apply`able things in Bel: primitives, closures, macros, and 
continuations.

To apply a primitive, we call `applyprim`. Let's skip down to it and 
see what it does. The first thing `applyprim` does is look the `prim` up 
in a list of `prims` to see if it really is one. If you're wondering 
why `prims` is organized the way it is, the primitives are grouped by 
the maximum number of arguments they take. This is frankly a trick to 
get around not having numbers yet; we can use udrop with the position
of the `prim` in `prims` to detect calls with too many arguments.

Since primitives can take at most two arguments, we pick out two, 
which we call `a` and `b`. Since `car` and `cdr` of `nil` both yield `nil`, this 
is how we make missing arguments to primitives default to `nil`. 

```lisp
> (join)
(nil)
```

If the call is a legal one, we call the primitive with the same name.
This is another place where behavior leaks from the instance of Bel 
in which the interpreter is running to the code it's evaluating: when 
the interpreter sees a call to car in the code it's evaluating, the
interpreter calls car itself.

The primitive might signal an error, if for example we call `car` on an 
atom. So the case expression containing the calls to the primitives
is wrapped in an `eif`, and if there's an error, we leak the same 
error object (whose nature is unspecified) up into the program we're 
evaluating. Otherwise we just "return" whatever the call returns, by 
consing it onto the return stack.

Since primitives are one of the cases where it's most confusing to 
have things happening at two levels, I want to be as clear as 
possible about how they work. The interpreter can call `car`. That's 
true by assumption. Up in the program being evaluated, we want `car` to
have a value. So we give it a sort of placeholder value, the list
`(lit prim car)`, which means, in effect, "please call `car` for me, 
interpreter," which the interpreter does whenever it encounters one
of these placeholders being called like a function.

Now back up to `applylit`. The next clause handles all the functions
that aren't prims, and it's implemented by calling `applyclo`. But
since functions are just lists and there's nothing to stop someone
from trying to call an ill-formed `clo`, we have to check the validity 
of every component of this supposed closure first.

If it checks out, we send it and the arguments to `applyclo`. This
function does two things, in the sense of putting two things on the 
expression stack. The first `fu` on the stack calls pass to get a 
lexical environment representing the parameter bindings onto the top 
of the return stack, and the second simply puts the body of the 
function (which is a single expression) on the stack, with those 
bindings as its environment.

The real work is done by `pass` and its subroutines `typecheck` and 
`destructure`. Between them these take a parameter list and a list of 
arguments, and turn them into an environment consisting of 
`(var . val)` pairs.

Before we look at them I should explain a feature of Bel parameter
lists that we haven't encountered yet: type checking. A parameter of 
the form 

```lisp
(t v f)
```

means that unless `f` is true of the value passed to `v`, an error will 
be signalled. So if we define

```lisp
(def consa ((t xs pair)) 
  (cons 'a xs))
```

and we call it with an argument that's not a pair, we'll get an 
error:

```lisp
> (consa 'z)
Error: mistype
```

If the variable and test in a `t` parameter are symbols, you can simply 
conjoin them with a vertical bar.

```lisp
(def consa (xs|pair)
  (cons 'a xs))
```

The reader expands `x|y` into `(t x y)`.

You can nest `t` parameters, `o` parameters, and parameters that are 
pairs without restriction. 

```lisp
(def foo ((o (t (x . y) [caris _ 'a]) '(a . b)))
  x)

> (foo '(b b))
Error: mistype
> (foo)  
a
```

Though as you can see, programs get hard to read if you do too much 
computation in the parameter list.

The way `pass` works is by descending depth-first through the 
parameter list, trying to match it up with the arguments. It takes
six arguments. The last three are the familiar `(s r m)`, and the 
first three represent a part of the parameter list, the 
corresponding part of the argument list, and the environment we've 
accumulated so far.

The reason we have to give `pass` `(s r m)` as well is that the work it 
does can't be done as a simple subroutine. It's not simply matching 
parameters and arguments. The default expressions for optional 
parameters mean we're really running the interpreter when we call 
`pass`. And `pass` doesn't return the result of matching parameters with
arguments, but rather ensures that one ends up on the top of the 
return value stack. (In other words, it causes a value to be returned 
in the Bel program that the interpreter is evaluating, rather than 
returning a value within the Bel instance in which the interpreter is 
running.)

It starts by defining a local function, `ret`, that returns a value in
the sense of calling `mev` with the new value consed onto the return 
stack. Then we have a bunch of cases to consider. If there are no 
parameters left, then we check whether there are any arguments left. 
If there are, the function has been called with too many arguments, 
and we should signal an error; if there aren't, then we've finished, 
and we can put the environment we've built onto the return stack.

If we get past that test, we know `pat`, the parameter or parameters,
is non-nil. Is it a literal? If so that's an error, because you
can't use a literal (other than `nil`) as a parameter. Whereas if it's 
a variable, we can just add a new `(var . val)` pair of it and whatever 
the arguments are to the env we've built so far, and "return" it 
using `ret`.

Since `literal` and `variable` between them cover all atoms, if we get 
past these tests we know `pat` must be a pair, so there are three
possibilities left: that's it's a `t` parameter, that it's an o`
parameter, or that it's an ordinary list.

If it's a `t` parameter, we call `typecheck`, and what that does is put a 
call to the test function on the expression stack, followed by a 
closure that checks whether that call returned true, and if so
recursively calls `pass` with the var stripped of the type check.
Otherwise it signals an error.

Whereas if `pat` is an optional parameter, since we know we have a
value for it (whatever arg is), we can ignore the default expression
and call pass recursively as if it were an ordinary parameter.

How do we know we have a value for `pat`? What if `arg` is `nil`? This
is not the point where we check whether we've run out of arguments.
If `arg` is `nil` at this point, that means the value being passed as an 
argument is `nil`, not that there are no arguments.

In the final case in `pass`, the parameter is a list, and we need to 
call `destructure` to go inside it and the arguments to see if they
match. When you call a function with a parameter list (rather than a 
parameter like `args` that's a single variable), this is the case you 
go through the first time through `pass`. 

This is where you notice if you've run out of arguments, and that is 
the first thing `destructure` checks. If the first parameter is an 
optional parameter, you're ok so far, because you can evaluate its 
default expression to get a value for it. In that case put the 
default expression on the stack, followed by a pair of closures that 
in effect do a depth-first traversal of the parameter list, one 
calling `pass` on the first parameter (with the value returned by the 
default expression as the "argument"), and a second calling `pass` on 
the remaining parameters, with `nil` as the argument, since we already 
know it's empty.

If `arg` is `nil` and the first parameter in our list isn't an optional
one, then we know the function was called with too few arguments,
and we signal an error.

```lisp
> ((fn (x y) x) 'a)
Error: underargs
```

The other error case is when `arg` is a non-nil atom. We can match such 
an argument with a single parameter, but not with a list of 
parameters. So if `arg` is an atom, we signal an error.

```lisp
> ((fn ((x y)) x) 'a)
Error: atom-arg
```

The default clause of `destructure` is the "normal" case where we have 
a non-empty list of arguments. Here we also use `mev` to do a 
depth-first traversal of the parameters, very much like the one we 
saw in the first clause of `destructure`.

The fact that `pass` uses the environment it's in the process of
building as the environment when it puts expressions on the stack
means the bindings of earlier parameters are visible to the code in 
later ones:

```lisp
> ((fn (x (o y x)) y) 'a)
a
> ((fn (f x|f) x) pair 'a)
Error: mistype
```

Now to continue with our own depth-first traversal of the 
interpreter, back up in `applylit`, which we're halfway through. The 
next clause is for macros. The difference between applying a macro 
and an ordinary macro call is that when you apply an operator, the 
arguments have already been evaluated. But you can generate a macro
call easily by `cons`ing the macro onto the front of a list of quoted 
arguments. So you just call `applym` on that. (You could send it to 
`mev`, but you know it will end up in `applym`, so you might as well go 
straight there.)

The next clause in `applylit` is for continuations. After checking the
validity of a continuation, we call `applycont` to call it.

In the definition of `ccc` we saw that making a continuation is simple: 
it's just a list of `s` and `r`. The work is done when the continuation 
is called. But even then it's not that much work. If a continuation 
is 

```lisp
(lit cont s2 r2)
```

you could call it on an argument a simply by saying

```lisp
(mev s2 (cons a r2) m)
```

except for two things. One is that you have to check that the form of 
the continuation is valid, just as you do with a closure. The other 
is that there may be `prot` entries in your current evaluation stack, 
and the whole point of prots is that those expressions get evaluated 
even if evaluation is interrupted before you reach them.

So you can't just use `s2` as the first argument to `mev`. You have to
combine `s2` with everything that you need to keep from your current 
expression stack. That includes `prot`s and also `bind`s, since the code 
in a `prot` could refer to a dynamically bound variable.

Those two possibilities account for all the complexity of calling
continuations. Otherwise it would be one line in `applylit`.

Finally, in the last clause of `applylit`, we see the code that makes
it possible, for example, to use a number in functional position:

```lisp
> (2 '(a b c))
b
```

The variable `virfns` (for virtual functions) is a list of `(n . f)` 
pairs where `n` is a type of lit and `f` is a function that generates an 
expression to be evaluated when such a lit is encountered in
functional position. Since these functions generate expressions,
they're a lot like macros.

Although we haven't introduced numbers yet, here's the virtual 
function for them, so you can see what one looks like:

```lisp
(vir num (f args)
  `(nth ,f ,@args))
```

It looks like a macro definition, and does much the same thing.  It 
will turn 

```lisp
(2 '(a b c))
```

into

```lisp
(nth 2 '(a b c))
```

There's a good deal more of the language still to come, including
numbers and I/O, but we have now seen the entire Bel interpreter.

Next come several functions for operating on functions. The first,
`function`, tells whether something is a function and if so what kind.

```lisp
> (map function (list car append 'foo))
(prim clo nil)
```

The next, `con`, returns a constant function:

```lisp
> (map (con 'yo) '(a b c))
(yo yo yo)
```

Then comes an important one, `compose`, which takes any number of
functions and returns their composition. For example, `cadr` is 
equivalent to a composition of `car` and `cdr`:

```lisp
> ((compose car cdr) '(a b c))
b
```

When the names of what you're composing are symbols or numbers, Bel 
lets you abbreviate calls to compose with colons. 

```lisp
> (car:cdr '(a b c))
b
```

Functional composition lets you eliminate a lot of unnecessary 
variables. I warned earlier that the definitions that occur early in 
the Bel source are not in truly idiomatic Bel, because that would 
require things that hadn't been defined yet. Composition is one of 
them. If you had to define `cadr` in idiomatic Bel you'd say

```lisp
(set cadr car:cdr)
```

There's a special abbreviation for composing `no`, which is to prepend 
a tilde.

```lisp
> (map ~cdr '((a) (a b c) (a b)))
(t nil nil)
```

Now we can understand the idiomatic definition of all that I gave 
early on:

```lisp
(def all (f xs)
  (~some ~f xs))
```

It's actually inaccurate to say that `compose` operates on functions,
because it can operate on anything that we can call like a function.

```lisp
> (2:or nil '(a b c))
b
```

I could solve this problem by inventing some new term for things
that can be called like functions, but that seems a bit bogus, so
instead I'm going to be irresponsible and just talk about functions
when I mean things callable like functions. Because the fact is that 
just about anywhere you can use a function, you can use anything else 
that's callable. 

The next function, `combine`, produces something more general: it takes 
one argument, `f`, and returns a function that combines functions using 
`f`. So this for example 

```lisp
((combine and) car cdr)
```

will yield a function equivalent to

```lisp
(fn (x)
  (and (car x) (cdr x)))

> (map ((combine and) car cdr)
       '((a . nil) (a . b) (nil . b)))
(nil t nil)
```

Since combine of `and` and `or` are so frequently needed, we predefine
two functions, cand and cor, for those cases.

```lisp
> ((cand pair cdr) '(a b))
(b)
> ((cor char pair) 'a)
nil
```

Next come the two classic reduction functions, left and right fold.
Once again, their definitions are complicated by taking any number of 
arguments, and they're easier to understand if we start with 
three-argument versions:

```lisp
(def foldl3 (f base xs)
  (if (no xs)
      base
      (foldl3 f (f (car xs) base) (cdr xs))))

(def foldr3 (f base xs)
  (if (no xs)
      base
      (f (car xs) (foldr3 f base (cdr xs)))))
```

When we look at these versions it's clear that left fold works by
growing the base case and then returning it when we get to the end of 
the list, and right fold works by growing a recursive call tree that 
builds its value on the way back up.

Here is a series of equivalent calls showing how a left fold plays 
out:

```lisp
(foldl3 cons nil '(a b))
(foldl3 cons (cons 'a nil) '(b))
(foldl3 cons (cons 'b (cons 'a nil)) nil)
```

This yields `(b a)`. The corresponding right fold,

```lisp
(foldr3 cons nil '(a b))
(cons 'a (foldr3 cons nil '(b)))
(cons 'a (cons 'b (foldr3 cons nil nil)))
```

yields `(a b)`.

For comparison, the `reduce` function we already have is a right fold 
where the base case is the last element of the list. 

The next function, `of`, is for situations like the following:

```lisp
(+ (car x) (car y) (car z))
```

With it you can write

```lisp
((of + car) x y z)
```

And `upon` is a sort of reverse call, in the sense that it saves an 
argument, and then tells you what calling things on that argument 
will return:

```lisp
> (map (upon '(a b c)) 
       (list car cadr cdr))
(a b (b c))
```

The next function, `pairwise`, returns true iff its first argument is 
true of every two-element window in the second. For example

```lisp
(pairwise f '(a b c d))
```

is equivalent to

```lisp
(and (f 'a 'b) (f 'b 'c) (f 'c 'd))
```

That expression

```lisp
(all [id _ (car args)] (cdr args))
```

in the definition of `=` would have been

```lisp
(pairwise id args)
```

if we'd had `pairwise` then.

The next function, `fuse`, takes a function that's expected to return 
a list, plus one or more lists, and returns the result of appending 
together the results of calling the function on successive elements 
of the lists. Which description incidentally demonstrates the 
advantage of source code over natural language in a spec.

```lisp
> (fuse [list 'a _] '(1 2 3))
(a 1 a 2 a 3)
```

We use it in `letu`, which is for creating uvars. In `eif` we had to say 

```lisp
(with (v (uvar)
       w (uvar)
       c (uvar))
  ...)
```

With letu that becomes

```lisp
(letu (v w c)
  ...)
```

You can also make one `uvar` by using a single variable instead of a
list of variables.

We use `letu` in `pcase`, which is like a case statement but with 
predicates as the keys.

```lisp
> (map [pcase _
         no   'empty
         atom 'atom
              'pair]
       '(a nil '(b c)))
(atom empty pair)
```

The next function, `match`, is like `=` except that functions occurring
in the second argument are treated as predicates, and ts `match`
everything.

```lisp
> (match '(a (b) c d) (list 'a pair 'c t))
t
```

The `split` function breaks its second argument at the point where its
first argument returns true.

```lisp
> (split (is \a) "frantic")
("fr" "antic")
```

The next two macros are for conditionally evaluating blocks of code: 
when's body is evaluated when the test is true, and unless's when 
it's false.

Now we come to the code that implements numbers. Before we continue
I'd like to remind the reader of the principle that when something
is defined in the Bel source, the goal is to specify what it means 
and not to provide an efficient implementation.

Numbers in Bel take the form

```lisp
(lit num (sign n d) (sign n d))
```

where the first `(sign n d)` is the real component and the second the
imaginary component. A sign is either `+` or `-`, and `n` and `d` are unary 
integers (i.e. lists of `t`) representing a numerator and denominator.

So 2/3 is 

```lisp
(lit num (+ (t t) (t t t)) (+ () (t)))
```

and `4-1/2i` is

```lisp
(lit num (+ (t t t t) (t)) (- (t) (t t)))
```

You'll never see this representation unless you go looking for it, 
because Bel reads and writes numbers in the conventional way.

```lisp
> (lit num (+ (t t) (t t t)) (+ () (t)))
2/3
```

We're going to build numbers up from the bottom, starting with 
operators for unary integers and finally defining the operators one 
actually uses in programs.

We start with a selection of constant unsigned integers. Then come a 
set of operators for them. Operators for unsigned integers have names 
beginning with `i`. Notice how with unary numbers, arithmetic 
operations turn out to be familiar list operations. E.g. `+` is append, 
and `*` is a fold of a fuse. One quirk: since these numbers are 
unsigned, `i-` has to return two values: the result and an indication 
of whether it's positive or negative.

The next level of numbers is unsigned rationals, which are `(n d)`
pairs of unsigned integers. Functions for operating on them begin 
with `r`.

The next level up from that is signed rationals, which are `(s n d)`,
triples where `s` is `+` or `-` and `(n d)` is an unsigned rational. 
Operators on signed rationals begin with `s`.

The top level, the numbers that programs actually use, are complex
numbers. These have the form shown above. Operators for complex
numbers have names beginning with `c`. 

Now that we've defined numbers, we get a series of functions for
making them, recognizing them, and taking them apart.

The one nontrivial one is simplify, which simplifies a rational by
dividing both components by their greatest common factor. This is 
called by `buildnum` after reading numbers and after every arithmetic 
operation.

After `buildnum` and `recip` we see the definitions of the `+`, `-`, `*`, and `/` 
functions that are actually used in programs. 

```lisp
> (+ .05 (/ 19 20)) 
1
```

Now that we have numbers we can define some common functions that use 
them, like `len` for finding the length of a list

```lisp
> (len "foo")
3
```

and `pos` for finding the position of something in one

```lisp
> (pos \a "ask")
1
```

(In Bel, lists are one-indexed.) 

We use `pos` in `charn`, which returns a unique integer for each
character based on its position in chars, a list of all the 
characters.

```lisp
> (map charn "abc")
(97 98 99)
```

We can also define functions for comparing numbers, and more 
generally for comparing things that can be transformed into numbers 
or lists of them.

The comparison operators used in programs are the familiar `<` and `>`. 
These are defined in terms of `bin<`, which compares two objects by 
looking up a comparison function for them. 

The comparison functions for different kinds of objects are kept in 
`comfns`, which is a list of pairs of functions `(f . g)` where `f` is a 
predicate (e.g. `real`) and `g` is a function of two arguments that 
returns true iff the first is less than the second.

Comparison functions are defined using `com`, and following its
definition we see the four predefined comparison functions, for 
reals, chars, strings, and symbols. So `<` and `>` work for arguments of 
those types,

```lisp
> (> 3/4 2/3)
t
> (< 'apple 'apply)
t
```

and if you use `com` to define a comparison function for other kinds of 
objects, `<` and `>` will work for them too.

Notice that `bin<` has an explicit test for nil arguments. That's
because `nil` is both a symbol and a string (the empty string).

Then come a few more predicates on numbers which are mostly used in 
type checking -- `int`, `whole`, and `pint` (positive integer). 

Next comes a familiar name, `yc`. This is the Y combinator, which is 
used to generate recursive functions. It's used in `rfn` to make a 
recursive function with a name,

```lisp
> ((rfn foo (x) (if (no x) 0 (inc:foo:cdr x))) '(a b c))
3
```

and `rfn` in turn is used to define `afn`, which lets you refer to a 
function within itself as self.

```lisp
> ((afn (x) (if (no x) 0 (inc:self:cdr x))) '(a b c))
3
```

We use `afn` in `wait`, which takes a function and calls it till it
returns a non-nil value.

```lisp
> (set x '(nil nil a b c))
(nil nil a b c)
> (wait (fn () (pop x)))
a
> x
(b c)
```

(We haven't defined `pop` yet but it's the usual `pop`.)

The function `runs` takes a function and a list and breaks up the list 
into stretches for which the function returns true or false.

```lisp
> (runs pint '(1 1 0 0 0 1 1 1 0))
((1 1) (0 0 0) (1 1 1) (0))
```

Using `whitec`, which returns true of whitespace characters, we can use 
`runs` to pick the words out of a string:

```lisp
> (tokens "the age of the essay")
("the" "age" "of" "the" "essay")
```

But since `tokens` can take any function or object to treat as a break, 
you can use it for more than picking out words.

The function `dups` returns all the elements of a list that occur more 
than once in it.

```lisp
> (dups "abracadabra")
"abr"
```

The next function, `simple`, is a predicate true of atoms and numbers.
You need this for e.g. the base case of tree traversals.

The `do1` macro is like do except that it returns the value of the
first expression in its body rather than the last. We'll see an 
example soon when we get to the queue functions.

The `consif` function `cons`es something onto the front of a list only if 
it's non-nil.

```lisp
> (consif (cadr '(a)) '(x y))
(x y)
```

The `check` macro evaluates an expression and returns its value only if 
its second argument returns true of it. 

```lisp
> (check (car '(1 2 3)) odd)
1
> (check (car '(1 2 3)) even)
nil
```

The optional third argument is an expression to evaluate to get an
alternate value in case the function returns false.

```lisp
> (check (car '(1 2 3)) even 2)
2
```

The `withs` macro is like `with` except that the bindings of the
preceding variables are visible in the expressions defining the
values of later ones.

```lisp
> (let x 'a
    (withs (x 'b
            y x)
      y))
b
```

The `bind` macro is like a call to `dyn` with a body. Since `bind` is a 
generalization of `dyn`, you would not ordinarily use `dyn` in programs.

The `atomic` macro uses bind to bind `lock` during the evaluation of its 
body. As we saw earlier in `mev`, the current thread won't be 
interrupted so long as `lock` is dynamically bound in it to a non-nil
value. So if you have a series of expressions that you don't want 
interrupted because they'd leave something in an inconsistent state,
you can protect them by wrapping an `atomic` around them.

The `tail` function returns the rest of its second argument from the 
point where its first argument returns true of it, if any.

```lisp
> (tail [caris _ \-] "non-nil")
"-nil"
```

The function `dock` takes the last element off a list

```lisp
> (dock '(a b c))
(a b)
```

while `lastcdr` returns the last non-nil `cdr` of a list

```lisp
> (lastcdr '(a b c))
(c)
```

and `last` uses it to get the last element.

```lisp
> (last '(a b c))
c
```

Now come three functions for manipulating queues. A queue is 
represented as a list within a list. That way several variables can 
be set to the same queue. 

If you modify the middle of a list that is the value of several
variables, the modification is shared:

```lisp
> (set x '(a b c) y x)
(a b c)
> (set (cadr x) 'z)
z
> y
(a z c)
```

But there is no way to put something on the front of a list in a way 
that will be shared by multiple variables that were set to it. We 
haven't gotten to `push` yet, but I'll use it anyway to make this 
clear. When you push something on the front of a list, you're
modifying the value of a variable, not the list structure itself.

```lisp
> (push 'hello x)
(hello a z c)
> x
(hello a z c)
> y
(a z c)
```

To modify a value that several variables share, you have to be able to 
translate it into changing the `car` or `cdr` of some existing pair. And 
if the shared value is a list within a list, you can.

```lisp
> (set x '((a)) y x)
((a))
> y
((a))
> (push 'hello (car x))
(hello a)
> y
((hello a))
```

That's why queues are represented as lists within lists.

You make a new queue with `newq`, put something in a queue with `enq`,
and take something off with `deq`.

```lisp
> (set x (newq))
(nil)
> (enq 'a x)
((a))
> (enq 'b x)
((a b))
> (deq x)
a
> x
((b))
```

Notice `enq` and `deq` use atomic. Otherwise you could get errors if code
in another thread did something to a queue while you were in the 
middle of doing something to it.

Next we see the `set` macro itself. This calls `hug` to assemble the 
implicit pairs of arguments, and then for each pair `(p e)` generates 
an expression in which a `where` is wrapped around `p`, and then the 
appropriate half of the resulting pair is set to the value of `e`. 
So e.g. 

```lisp
(set (car x) 'a)
```

is expanded into the equivalent of

```lisp
(atomic (let v 'a 
          (let (cell loc) (where (car x) t)
            ((case loc a xar d xdr) cell v))))
```

We also have a more general macro, `zap`, that modifies the value of
something by applying a function to it.

```lisp
> (let x '(a b c)
    (zap cdr x)
    x)
(b c)
```

It's instructive to compare the definitions of `set` and `zap`. They're
much the same, except that `zap` has to wrap a call to apply around the
current value of the appropriate half of the pair returned by where. 

One subtle difference: the second argument to `where` in `set` is `t`, 
whereas in `zap` it's omitted (and thus defaults to `nil`). That's 
because setting an unbound variable should create a binding for it, 
but zapping one shouldn't.

With `zap` it becomes easy to write a whole class of other macros we 
need, like `++` and `--,` which increase and decrease the value of 
something

```lisp
> (let x '(1 2 3)
    (++ (car x) 10)       
    (-- (cadr x))
    x)
(11 1 3)
```

and `push` and `pull`, which put something onto the front of a list and 
remove something from one

```lisp
> (let x '(a b c)
    (push 'z x)
    (pull 'c x)
    x)
(z a b)
```

Since `pull` passes the optional comparison function to `rem`, we can say

```lisp
> (let x '(7 3 0 9 2 4 1)
    (pull 4 x >=)
    x)
(3 0 2 1)
```

I think this sort of "porosity" is in general a good idea. There's a 
case to be made for treating functions as black boxes, but with 
macros you often want to acknowledge what a macro expands into and 
pass through its optional arguments if you can.

Now we have all the operators we need to write the Bel reader. This
is like an inverted pyramid built upon the `rdb` primitive, which reads 
a single bit.

We want to be able both to read a character from a stream and also to 
look at the stream to see what character is waiting to be read. We 
simulate that with a global list of "unconsumed" characters, one for 
each open stream, kept as a list of `(stream . char)` pairs in `cbuf`. 
At first the only open input stream is the initial one, represented 
by `nil`, so `cbuf` starts out with one entry, `(nil . nil)`, which is 
`(nil)`.

After `cbuf` we see open and close, which call the `ops` and `cls`
primitives respectively, but also create and delete `cbuf` entries for
the streams involved.

Next comes `peek`, which tells you the next character waiting to be 
read from a stream. As well as streams, the Bel I/O functions take 
queues of characters (i.e. strings within lists), and that's what 
the first test in `peek` is checking for. If `s` is a queue, `peek` just 
returns the first element. Otherwise it begins by checking `cbuf` to 
see if it contains a character for this stream. If it does, peek 
returns it. Otherwise it calls `bitc` to try to read a new character.

The `bitc` function is where bits returned by `rdb` get turned into 
characters. It accumulates bits in `bbuf`, which is a list of 
`(stream . bits)` pairs. Once bits `=` the representation of some 
character in chars, `bitc` returns that character. 

The `rdb` primitive is non-blocking, meaning if no bit is currently 
available from the stream passed to it, it returns `nil` instead of 
waiting for one. Since `bitc` does the same, returning `nil` immediately 
if it gets `nil` back from `rdb`, it's non-blocking too. But the `wait` 
expression in `peek` means it won't return till `bitc` gives it a 
character or `eof`. If `peek` gets an `eof`, it returns `nil`.

Next comes `rdc`, which is like `peek` but consumes the character it 
returns. I.e. if a source has characters waiting to be read, repeated 
calls to `peek` will return the same character, and repeated calls to 
`rdc` will return successive ones.

```lisp
> (let s '("abc")
    (list (peek s) (peek s)))    
"aa"
> (let s '("abc")
    (list (rdc s) (rdc s)))
"ab"
```

The structure of `rdc` is similar to that of `peek`. First check if the 
argument is a stream. If it is, either return whatever's stored in 
`cbuf` (wiping the `cbuf` entry in the process) or call `bitc`.

Like `peek`, `rdc` waits till `bitc` gives it a character or `eof`, and 
returns `nil` in the latter case.

Then come some functions for recognizing types of characters.
Whether a character is a digit depends on the base, which is why 
`digit` takes an optional base argument.

```lisp
> (digit \a)
nil
> (digit \a i16)
t
```

The next function, `breakc`, returns true iff a character signals the 
end of a symbol or number. Among the characters treated as `breakc`s 
are those used as keys in syntax; we'll see soon which those are.

The `signc` function returns true iff its argument is a numeric sign,
and `intrac` is for detecting characters that have a special meaning 
if they occur within symbols. I'll explain what they do when we get 
to that part of the reader.

The `source` function returns true iff its argument is something that
can be read from, i.e. either `nil` (representing the initial input
and output streams), an object of type stream, or a queue of 
characters.

Then comes `read` itself. All its arguments are optional: a source to
`read` from, which defaults to `ins`, the default input stream; a base to 
read numbers in, which defaults to 10; and a value to return in case 
of `eof`, which defaults to `nil`.

In `read` we see the first parameters with type restrictions; the first 
argument has to be a source, and the second has to be between 2 and 
16 inclusive. You can use type checking however you like, but I 
personally use it only to catch errors that won't be caught 
otherwise.

Then comes `saferead`, which is like `read` but returns an alternate
value if there's an error or we get an `eof` from the stream.

Finally comes the actual reader, `rdex`, which is to reading what `ev` 
is to evaluation. There is one new parameter in `rdex`: `share`, which is 
a list of shared structure we've encountered so far.

Sometimes it matters when the same pair occurs multiple times in a 
tree. For example, when we used `smark` to mark special stack entries 
in the interpreter, it mattered that the cars of such entries were `id` 
to `smark` and not merely `=` to it. The Bel printer and reader use 
labels of the form `#n`, where `n` is whole number, to mark shared pairs. 
For example:

```lisp
> (let x '(a) 
    (list x x))
(#1=(a) #1)
> (set x '(a #1=(b) #1 c))
(a #1=(b) #1 c)
> (id (2 x) (3 x))
t
```

The `share` parameter keeps track of shared objects we've read so far, 
and their labels. It's passed to most functions in the reader, and 
most return two values: whatever has been read, and the `share` list.

The first thing `rdex` does is call `eatwhite` to consume any whitespace 
before the object it's trying to read. There are two kinds of 
whitespace, corresponding to the two clauses in `eatwhite`: invisible 
characters like spaces and linefeeds, and comments, which begin with 
semicolons and continue till the end of the current line. In the 
latter case we call `charstil` to consume all the characters up to the 
next linefeed.

When it has finished reading whitespace, `rdex` can do one of three
things depending on what `rdc` sees next. If `rdc` returns `nil`, that
means the stream is out of characters, and we return whatever we've 
been told to return in case of `eof`. Otherwise the value returned by 
`rdc` is either a syntax character, like e.g. a left parenthesis or a 
double quote, in which case we have something special to do, or an 
ordinary character, in which case we call `rdword`, which reads a word
(roughly a symbol or number).

The syntax characters are for `rdex` what special forms are for `ev`. 
Like forms, syntax is a list of `(char . function)` pairs that 
describe what to do if each character is encountered. There is a `syn` 
operator, akin to `form`, that defines the behavior of each character. 
The first `syn`, for example, defines what happens when the reader 
encounters a left parenthesis: it calls `rdlist` to read the elements, 
stopping when it encounters a right parenthesis.

If we skip down to `rdlist`, we see that it too begins with an 
`eatwhite`. Then there are several cases depending on what predicate
is true of whatever's seen by `peek`. If there's no character, then we 
have a read error, because we at least need to see the list 
terminator (e.g. right parenthesis) before we're done. If it's a 
period, then we call `rddot` to handle that case. If it's the list 
terminator we were looking for, then we're done reading this list, 
which we've been accumulating in acc, and we can return it. 
Otherwise the character is the beginning of the next element of the 
list, so we call `rdex` to read it, and then call `rdlist` recursively 
with the new object snocced onto the end of `acc`.

Let's take a look at `rddot`, which is not simply for reading dotted 
lists, because we could also be seeing something (e.g. a decimal 
number) that begins with a period. Which it is depends on whether 
there's a `breakc` after the period. If there is, we call `hard-rdex` to 
read whatever object is in the `cdr` of the list. This function is 
like `rdex` but signals an error if it can't read anything.

If we look at the `syn` for a right parenthesis, on the other hand, all 
it does is call error. That's because a `syn` says what to do when 
you're starting to read something, and since `rdlist` always reads all 
the way to the matching right parenthesis, if you encounter a right 
parenthesis when you're starting to read something, what you're 
seeing must be an unbalanced parenthesis.

There's a similar pattern for `\[` and `\]`, except that we do something 
with the list we've read: we make it into a fn expression.

```lisp
> (read '("[cons _ 'b]"))
(fn (_) (cons _ (quote b)))
```

The sym for `\\` shows how characters are represented. A backslash can 
either be followed by the character itself, or the name of one. The
names of characters like `\bel` that have them are kept in namecs.

Next come the `syn`s for `quote`, `backquote`, and `comma`. They all call
`rdwrap`, which does a `hard-rdex`, and then wraps the corresponding
operator around it.

Then comes the `syn` for `\"`, which calls `rddelim` to read a sequence of 
characters up to the next `\"`. Within the sequence of characters, a
backslash acts as an escape character, meaning whatever occurs next 
is treated as part of the sequence, even if it's the delimiter that 
would otherwise signal the end.

```lisp
> "foo\"bar"
"foo\"bar"
```

The escape character is not part of the string, just something used 
in its representation.

```lisp
> (mem \\ "foo\"bar")
nil
```

The character `\¦` acts as a delimiter for symbol names that could not 
otherwise be read. It works just like `\"` except that we call sym on 
the result.

```lisp
> '¦foo bar¦
¦foo bar¦
```

As we saw earlier, the character `\#` indicates shared list structure. 
If the `#n` occurs alone, it refers to a previously seen object; if 
it's followed by `=`, then it becomes the label for whatever is read 
next. 

In the latter case we call `rdtarget` to read it. This function does
some slightly odd things. It creates the pair `(cell)` to represent the
object it will read before it's read, then when it does read 
something `(e)`, it writes its `car` and `cdr` into the `car` and `cdr` of 
`cell`, and returns that. Why? Why not just return `e`? So that we can 
read lists that contain themselves. If we want to be able to read 
something like

```lisp
#1=(a #1 b)
```

we need to add it to the list of shared objects before reading it. Or 
more precisely, add to the list of shared objects a pair that we will 
later turn into it.

Now we come to `rdword`, which reads either a number, a symbol, or 
several numbers or symbols joined by intrasymbol characters like `\:`.

Unlike many other parts of the reader, this code knows from the start 
which characters it's dealing with: everything up to the next `breakc`. 
It first tries to parse those characters as a number, by calling 
`parsenum`. If that fails, the next level of precedence is the vertical 
bar used in type specs. There is nothing specific to parameter lists 
about this bar, incidentally; it's just something the reader expands 
into a list beginning with `t`.

```lisp
> 'for|2  
(t for 2)
```

If there's no vertical bar, we look for the intrasymbol characters
`\.` and `\!`. These are expanded by the reader into common expressions.

`a.b` expands into `(a b)`

`a!b` expands into `(a 'b)`

So for example if you wanted to insist that a parameter was a 
continuation, instead of expressing the parameter as

```lisp
(t c (isa 'cont))
```

you could just write

```lisp
c|isa!cont
```

and our recent example

```lisp
(id (2 x) (3 x))
```

could have been written 

```lisp
(id 2.x 3.x)
```

I haven't used this notation so far in the Bel spec, but it's quite
convenient. Any combination of `\.` and `\!` is allowed.

```lisp
> 'a!b.c
(a (quote b) c)
```

If there is nothing before the first `\.` or `\!`, the missing symbol is
treated as an implicit `upon`.

```lisp
> '!a
(upon (quote a))
> (let x '(a . b)
    (map .x (list car cdr)))
(a b)
```

The next level of precedence, represented by `parsecom`, is intrasymbol 
colons. These, as we've seen, are abbreviations for compose 
expressions.

```lisp
> 'a:b:c
(compose a b c)
```

The next level of precedence is prepended tildes, which as we've also
seen are abbreviations for `no` expressions. Here's some of each:

```lisp
> 'x|~f:g!a
(t x ((compose (compose no f) g) (quote a)))
```

The lowest level of precedence is the else clause of `parseno`, where
we try to parse a number, and failing that (since there can now be no 
intrasymbol characters in it), treat whatever we've got left as a 
symbol.

That's the end of the reader. Next comes the definition of the `bquote` 
macro. This is not part of the reader per se; the syns for `\`` and `\`, 
merely yield bquote and comma expressions; but it feels like a part 
of the reader in the sense that you never use bquote or comma 
expressions directly.
 
The `bquote` macro is the biggest in the Bel source, and gives a taste 
of how macros, as they become more powerful, can become like 
compilers.

The general goal of backquote is to turn an expression containing
backquotes, commas, and comma-ats into calls to the familiar 
list-building functions cons and append. To get an idea of what we're 
aiming at, here are some backquoted expressions followed by their 
expansions:

```lisp
  `(x ,y)          (cons (quote x) (cons y nil))

  `(x ,@y)         (cons (quote x) (append y nil))

  `(,x ,@y)        (cons x (append y nil))

  `(,@x ,@y)       (append x (append y nil))

  `(x . ,@y)       (apply cons (quote x) y)

  `(,@x . ,@y)     (apply append x y)
```

These are not quite the expansions generated by `bquote`. I've cut some 
error-checking code to make them clearer.

Much of the complexity of `bqex` is due to the need to handle nested
backquotes. We'll find it easier to understand `bqex` if we start with
a version that doesn't:

```lisp
(def bqex1 (e)
  (if (no e)   (list nil nil)
      (atom e) (list (list 'quote e) nil)
               (case (car e)
                 comma    (list (cadr e) t)
                 comma-at (list (list 'splice (cadr e)) t)
                          (bqexpair1 e))))

(def bqexpair1 (e)
  (with ((a achange) (bqex1 (car e))
         (d dchange) (bqex1 (cdr e)))
    (if (or achange dchange)
        (list (if (caris d 'splice)
                  (if (caris a 'splice)
                      `(apply append (spa ,(cadr a)) (spd ,(cadr d)))
                      `(apply cons ,a (spd ,(cadr d))))
                  (caris a 'splice)
                  `(append (spa ,(cadr a)) ,d)
                  `(cons ,a ,d))
              t)
        (list (list 'quote e) nil))))
```

Now it's easier to follow what will happen to an expression as it
gets expanded.

The `bqex1` function returns two values: an expansion, and a second
value indicating whether anything changed in generating it. For
example, if we expand an expression containing no commas or 
comma-ats, nothing need change.

```lisp
> (bqex1 '(x y z))
((quote (x y z)) nil)
```

That call will go through the last clause of `bqex1`, and then, since 
the recursive calls to `bqex1` on the `car` and `cdr` at the start of 
`bqexpair1` will get nil values for both `achange` and `dchange`, we'll go 
through the last clause of `bqexpair1` as well. 

Now suppose we do have a `comma` or `comma-at` in our expression. The
base cases of `bqex1` are for when the whole of `e` is a comma or 
comma-at expression. When it's a `comma` expression, the expansion is 
simply its argument.

```lisp
> (bqex1 '(comma x))
(x t)
```

But when it's a `comma-at` expression, the expansion is a call to 
splice:

```lisp
> (bqex1 '(comma-at x))
((splice x) t)
```

What's `splice`? If we look at its definition, it simply signals an 
error. Why generate an expansion that signals an error? Because if 
the expression we're expanding is a naked `comma-at`, meaning one 
that's not within a list, we should signal an error; there's nothing 
to splice the value into.

```lisp
> (let x '(a b c)
    `,@x)
Error: comma-at-outside-list
```

Now let's see what happens when we have `comma` and `comma-at`
expressions within a list. In that case we call `bqexpair1`, which
generates expansions for its `car` and `cdr`, and then puts them
together to make an expansion for the whole list.

Suppose we're expanding 

```lisp
`(,x . ,@y)
```

The `car` and `cdr` of this list are the two cases we just examined. 
I.e. within `bqexpair1`

```lisp
(a achange)
```

will be

```lisp
(x t)
```

and 

```lisp
(d dchange)
```

will be

```lisp
((splice y) t)
```

What happens when these get put together? Since the `car` of `d` is a
splice expression and the `car` of `a` isn't, we go through this clause

```lisp
`(apply cons ,a (spd ,(cadr d)))
```

yielding

```lisp
(apply cons x (spd y))
```

Notice that the `splice` put on the front of its return value by `bqex1` 
is taken back off when the return value gets back to `bqexpair1`. 
That's why expansions of comma-ats don't always signal errors; it's 
only a "leftover" splice that signals an error.

The call to `spd` is the error-checking code that I mentioned I cut
from the examples above. As long as `y` is the right sort of value, 
meaning a list of one element, `spd` simply returns it. But it also
signals an error if it's not.

```lisp
> (let x '(b c)
    `(a . ,@x))
Error: splice-multiple-cdrs
```

You can't splice anything except a one-element list into the `cdr` of
a pair. The case above is like writing `(a . b c)`, which is 
ill-formed.

Here's an example of this expansion in action:

```lisp
> (with (x 'a y '(b))
    `(,x . ,@y))
(a . b)
```

All the code that you see in the Bel source but not in `bqex1` is for
dealing with nested backquotes. The additional argument, `n`, is the
nesting level, represented by an actual nested list. The function
`bqthru` is for going through `comma` and `comma-at` operators to get at 
the ones within that need to be expanded, since the outermost
backquote matches the innermost comma or comma-at. We increment the 
nesting level (by saying `(list n)`) each time we go through a `bquote`, 
and decrement it (by saying `(car n)`) each time we go through a `comma` 
or `comma-at`. Until the nesting level is nil, whereupon we expand 
the comma or comma-at as in the examples above.

```lisp
> (bqex '(bquote (comma (comma x))) nil)
((list (quote bquote) (list (quote comma) x)) t)
```

Notice here for example we expand the innermost comma expression, but 
leave the outer one "intact," in the sense of turning it into a call
to `list` that makes a new comma expression.

Like the `splice` macro, the `comma` and `comma-at` macros signal errors
if they're ever called. In a well-formed backquote expression they
never do get called, because they're expanded away. They only get 
called when there are "leftover" commas or comma-ats that don't have 
matching backquotes.

```lisp
> (let x '(a b c)
       `,,x)
Error: comma-outside-backquote
```

Now we come to the printer. The `print` function takes an object and
an optional stream and prints the object to the stream. There are
two other optional parameters, `names` and `hist`, which programs are
unlikely to supply explicitly; the first is a list of shared 
structure, which as we saw in read has to be displayed in a special
way, and `hist` is a list of pairs printed so far.

There are four cases in `print`. If the thing we're printing is simple, 
meaning an atom or number, we call `prsimple` to print it. In `prsimple` 
there are four cases; two of them, chars and streams, are handled 
right there. 

In the clause for chars we're down to printing individual characters 
with `prc`. This in turn uses `wrb` to write the bits in the character's
binary representation (or simply enqueues the character, if printing 
to a queue).

Notice that `prsimple` doesn't print streams in a way that can be read 
back in, or even distinguished from one another. Streams have no 
readable representation. 

Nearly all the complexity in `prsymbol` is for dealing with symbols
that need to have `\¦`s around their names because otherwise they'd be 
unreadable. If it weren't for that possibility, `prsymbol` could be

```lisp
(map [prc _ s] (nom x))
```

The way we check if a symbol needs to have `\¦`s around its name is by 
calling read on the characters in its name. If we don't get the same 
symbol back, we need to wrap it in `\¦`s. In the middle of which we call
`presc` to print the characters in the symbol with backslashes before 
any `\¦`s.

To print a number we call `prnum` on its real and imaginary components.
This in turn calls `rrep` to generate fractional representations of 
unsigned rationals. Bel can read numbers in decimal format, but 
internally they're rationals and are displayed that way.

```lisp
> 1.5
3/2
```

Now back up to `print`, where the next clause is for printing strings.
This is complicated by the fact that strings are lists and can thus
share structure with other things we're printing. If `ustring` says
that our string is a unique one, meaning one that doesn't share 
structure with anything in names, then we can simply print it as a 
series of characters between double quotes (using `presc` in case there 
is a double quote in the string).

```lisp
> "foo"
"foo"
```

The next clause in print checks whether the thing we're printing is 
one of the shared objects in names. If it is, we have to display it 
using either `#n=...` if it's not in `hist` (meaning this is the first 
time we're printing it) or as `#n` if it is.

```lisp
> (let x '(a b c)
    (list x x))
(#1=(a b c) #1)
```

To print a pair we call `prpair`. This prints parentheses around a call 
to `prelts` that prints the contents of the pair. In `prelts` we begin by 
calling `print` recursively on the `car`. Then we have to decide how to 
print the `cdr`. If it's simple, or a unique string, or shared 
structure, we display the list as a dotted list.

```lisp
> (cons 'a 5)
(a . 5)
> (let x '(a b c)
    (cons x x))
(#1=(a b c) . #1)
```

Otherwise we print a space and call `prelts` recursively on rest.

```lisp
> (append '(a b c) 5)
(a b c . 5)
```

Finally come a couple of functions for printing in specific 
situations. The first is `prn`, which is used to display data, e.g. for 
debugging. It takes any number of arguments and prints them separated 
by spaces, followed by a newline.

```lisp
> (with (x "foo" y 'bar)
    (prn 'x x 'y y))
x "foo" y bar 
bar
```

It returns the value of the last, so you can wrap a call to `prn` 
around any expression in your code without changing what it returns.

The second, `pr`, is for displaying messages to humans. If you're
looking at data, you want to print strings in double quotes, but if 
you're printing a message to a user, you probably don't. E.g.

```lisp
(let user 'Dave 
  (pr "I'm sorry, " user ". I'm afraid I can't do that."))
```

will print

```lisp
I'm sorry, Dave. I'm afraid I can't do that.
```

It calls `prnice` to print its arguments in a readable form.

The remaining definitions in the Bel source aren't needed to write
the interpreter or `read` or `print`, but are useful in programs.

The `drop` function takes `n` elements off the front of a list

```lisp
> (drop 2 '(a b c d e))
(c d e)
```

and `nth` returns the `nth` element of one.

```lisp
> (nth 2 '(a b c d e))
b
```

As I mentioned earlier, lists in Bel are 1-indexed.

We `use` nth in the virtual function for numbers, which I also included 
earlier.

```lisp
> (2 '(a b c))
b
```

We see this in use in `nchar`, which is the converse of `charn`. It
converts a number to a character.

```lisp
> (nchar 65)  
\A
```

The `first` function is the complement of `drop`. It returns a list of 
the first `n` elements of a list.

```lisp
> (first 2 '(a b c d e))
(a b)
```

The `catch` macro is used to establish a continuation to which we can 
throw control if we want to abort a computation. It dynamically binds 
`throw` to the continuation, so whatever argument we give to `throw` will 
be returned as the value of the catch expression.

```lisp
> (catch 
    (throw 'a)
    (/ 1 0))
a
```

The `cut` function returns the elements of a list from its second
argument to its third inclusive,

```lisp
> (cut "foobar" 2 4)
"oob"
```

unless the second argument is negative, in which case it indicates
how many elements to take off the end of the list.

```lisp
> (cut "foobar" 2 -1)
"ooba"
```

The `whenlet` macro is to when as `iflet` is to `if`. It's used in `awhen`, 
which is to when as `aif` is to `if`.

The `each` macro causes its body to be evaluated with a variable bound 
to successive elements of a list.

```lisp
> (let x nil
       (each y '(a b c)
         (push y x))
       x)
(c b a)
```

Now come several functions on functions. The first, `flip`, takes a
function and returns a function that calls it with the parameters
reversed.

```lisp
> ((flip -) 1 10)
9
```

The next, `part`, is for partial application. It takes a function and
several arguments, and returns a function that will call it with 
those arguments plus whatever additional ones are supplied.

```lisp
> ((part cons 'a) 'b)
(a . b)
```

As its name suggests, `trap` is a backwards `part`; it puts the newer
arguments first.

```lisp
> ((trap cons 'a) 'b)
(b . a)
```

Finally, `only` takes a function and returns a function that calls it
only if its first argument is non-nil. So instead of

```lisp
(let x (f)
  (if x (g x)))
```

or 

```lisp
(aif (f) (g it))
```

you can say just

```lisp
(only.g (f))
```

So for example `find`, which we defined 

```lisp
(def find (f xs)
  (aif (some f xs) (car it)))
```

could with `only` and composition be defined

```lisp
(set find only.car:some)
```

Incidentally, these examples show the sort of situation where 
intrasymbol dot notation is useful. You wouldn't want to use it for 
every expression in a program, but it's readable when used with a 
"modifier" like `only`.

These are followed by some functions involving numbers, including
`>=` and `<=`, `floor`, which returns the largest integer `<=` its argument,
and `ceil`, which returns the smallest integer `>=` its argument.

```lisp
> (map (upon 3.5) (list floor ceil))
(3 4)
```

The `mod` function returns its first argument modulo its second,

```lisp
> (mod 17 3)
2
```

Then come a group of macros for iteration. The first, `whilet`, 
evaluates its body while a test expression returns true, with a 
variable bound to its value.

```lisp
> (let x '(a b c)
    (whilet y (pop x)
      (pr y))
    x)
abcnil
```

Then comes the general `loop` macro, which sets a variable to an
initial value, then updates the value on successive iterations, so 
long as some test expression is true.

```lisp
> (loop x 1 (+ x 1) (< x 5) 
    (pr x))
1234nil
```

Several other iteration macros can then be defined in terms of loop, 
starting with `while`

```lisp
> (let x '(a b c)
    (while (pop x)
      (pr x)))
(b c)(c)nilnil
```

The `til` macro evaluates its body with a variable bound to the value 
of one expression till another expression returns true.

```lisp
> (let x '(a b c d e)  
    (til y (pop x) (= y 'c)
      (pr y))
    x)
ab(d e)
```

While it might seem as if you could define `whilet` simply as

```lisp
(mac whilet (var expr . body)
  `(til ,var ,expr (no ,var) ,@body))
```

this would fail if `var` were a pair.

The `for` macro starts a variable at some initial value and then
increments it by 1 on each succeeding pass through the loop, till it 
exceeds some threshold.

```lisp
> (for x 1 10 
    (pr x))
12345678910nil
```

The `repeat` macro simply evaluates its body a certain number of
times. 

```lisp
> (repeat 3 
    (pr 'bang))
bangbangbangnil
```

And the `poll` macro evaluates an expression till a function is true of 
the result, at which point it's returned.

```lisp
> (let x '(a b c d e)
    (poll (pop x) is!c)
    x)
(d e)
```

The `accum` macro takes a variable and a body of code and evaluates the 
body with the variable lexically bound to a function that accumulates 
all the values it's called on. It returns the list of values.

```lisp
> (accum a
    (map (cand odd a) '(1 2 3 4 5)))
(1 3 5)
```

It's used in `nof`, which takes a number and an expression and returns 
the result of evaluating the expression that number of times.

```lisp
> (nof 10 (rand 10))
(9 7 6 2 9 1 7 0 0 0)
```

(Sequences of random numbers never look random enough.)

The `drain` function is a combination of `poll` and `accum`. It evaluates 
some expression till it returns `nil`, or a value passing some other 
optional test, and returns a list of all the values the expression 
returned.

```lisp
> (let x '(a b c d e)
    (drain (pop x)))
(a b c d e)
> (let x '(a b c d e)
    (drain (pop x) is!d))
(a b c)
```

The `^w` function returns a number raised to a whole power.

```lisp
> (^w 2+3i 3)
-46+9i
```

The `randlen` function returns a random number that can be represented
in n binary digits, which is the same as saying a random integer
between 0 and 2<sup>n-1</sup> inclusive. It works by generating such a digit 
string and calling `read` on it. Within `randlen` we see the first use of 
the `coin` primitive, which randomly returns `t` or `nil`. 

The `clog2` function returns the ceiling of the logarithm base 2 of its
argument. It and `randlen` are used to build the general `rand` function,
which takes a positive integer `n` and returns a random integer between
`0` and `n-1` inclusive.

Next come a group of macros for assignment, starting with `wipe`, which 
sets multiple things to `nil`.

```lisp
> (let x '(a b c d e)
    (wipe 2.x 4.x)
    x)
(a nil c nil e)
```

Then we get the familiar `pop` macro, which takes the first element off 
a list and returns it.

```lisp
> (let x '(a b c)  
    (list (pop x) x))
(a (b c))
```

Like `any` macro that uses where for assignment, `pop` can operate on any
pair that you can find a way to refer to.

```lisp
> (let x '(a b c)
    (pop (cdr x))
    x)
(a c)
```

The `clean` macro removes everything from a list for which its first
argument returns true:

```lisp
> (let x '(1 2 3 4 5)
    (clean odd x)
    x)
(2 4)
```

And `swap` rotates the values of its arguments 

```lisp
> (let (x y z) '(a b c)
    (swap x y z)
    (list x y z))
(b c a)
```

as a degenerate case of which, if there are two arguments, it swaps 
their values.

```lisp
> (let x '(a b c d e)
    (swap 2.x 4.x) 
    x)
(a d c b e)
```

The `adjoin` function conses something onto a list only if it's not 
already an element of it.

```lisp
> (adjoin 'a '(a b c))
(a b c)
> (adjoin 'z '(a b c))
(z a b c)
```

It's used in `pushnew`, which pushes an element on a list if it's not
already an element.

```lisp
> (let x '(a b c d e)
    (pushnew 'a x)
    (pushnew 'z x)
    x)
(z a b c d e)
```

The `dedup` function returns a version of list without the later
instances of duplicate elements.

```lisp
> (dedup "abracadabra")
"abrcd"
```

Next come a group of functions involving sorting. The first inserts 
an element into a (presumably sorted) list according to a comparison 
function `f`.

```lisp
> (insert < 3 '(1 2 4 5))
(1 2 3 4 5)
```

Then we get `sort` itself, which uses `insert` to sort a list.

```lisp
> (sort < '(5 1 3 2 4))
(1 2 3 4 5)
```

As much as possible of the original order is preserved.

```lisp
> (sort (of > len) '((a b) (c) (d e) (f)))
((a b) (d e) (c) (f))
```

The `best` function takes a function and a list and returns the
element of the list that beats the others according to the function.

```lisp
> (best > '(5 1 3 2 4))
5
```

In case of ties it will return the first.

```lisp
> (best (of > len) '((a) (b c) (d e) (f)))
(b c)
```

It's used by `max` to return the largest of its arguments

```lisp
> (max 3 7 2 1)
7
```

and by `min` to return the smallest.

```lisp
> (min 3 7 2 1)
1
```

Sometimes you want to use `max` and `min` on individual arguments and
sometimes on a list. If we define them for the former case we can get 
the latter with `apply`.

The `even` function returns true of even numbers, and `odd` of odd ones.

```lisp
> (map upon.3 (list even odd))
(nil t)
```

And `round` rounds its argument to the nearest integer, choosing even
integers in case of ties.

```lisp
> (map round '(-2.5 -1.5 -1.4 1.4 1.5 2.5))
(-2 -2 -1 1 2 2)
```

Next come a group of functions for operating on files. The first is a 
sort of `let` for files; it opens one, lexically binds the resulting 
stream to a variable within the body, then uses an after to make sure 
the stream gets closed. If we evaluate

```lisp
(withfile s "foo" 'out
  (print 'hello s)
  (prc \  s)
  (print 'filesystem s))
```

The file "foo" will contain

```lisp
hello filesystem
```

It's used in `from` and `to`, which dynamically bind `ins` and `outs` (the 
default input and output streams) to the newly opened file. So if 
we say

```lisp
(to "foo" 
  (map prn '(a b c)))
```

then "foo" will end up containing

```lisp
a
b
c
```

and thus

```lisp
> (from "foo"  
    (drain (read)))
(a b c)
```

The function `readall` uses the same technique to read all the objects 
in a file, except it uses a unique pair to signal eof so that it can 
return nil as well.

```lisp
> (from "foo"
    (readall))
(a b c)
```

The load function reads all the objects in a file and also evaluates 
them. This is how you load a file of code. 

```lisp
> (to "example.bel"    
    (print '(def foo (x) (cons 'a x))))
nil
> (load "example.bel")
nil
> (foo 'b)
(a . b)
```

The `record` macro is used for making a transcript. It binds outs to a
newly created queue, then at the end returns the characters it 
contains.

```lisp
> (record
    (pr 'what)
    (pr \ )
    (pr "he said"))
"what he said"
```

The function `prs` uses it to print its arguments to a string.

```lisp
> (prs 'what \ "he said")
"what he said"
```

Then come functions for operating on arrays. Bel arrays are
recursively defined in the sense that an n dimensional array is a 
list of n-1 dimensional arrays.

We can get a new array by giving array a list of dimensions and an 
optional default value.

```lisp
> (set a (array '(2 3) 0))
(lit arr (lit arr 0 0 0) (lit arr 0 0 0))
```

Now `a` is a 2x3 array of zeros. 

The `vir` for arrs means we can use arrays as if they were functions on 
lists of indices, as implemented by `aref`. Vectors are one-indexed.

```lisp
> (a 1 1)
0
```

If we supply fewer indices than the array has dimensions, we get an 
array back instead of a scalar.

```lisp
> (a 1)
(lit arr 0 0 0)
```

Like any reference to part of a pair, array references are settable.

```lisp
> (++ (a 1 1) 5)
5
> a
(lit arr (lit arr 5 0 0) (lit arr 0 0 0))
```

Including references to subarrays:

```lisp
> (swap (a 1) (a 2))
(lit arr 5 0 0)
> a
(lit arr (lit arr 0 0 0) (lit arr 5 0 0))
```

Next come a set of functions for operating on tables, which are 
key-value stores.

We make a new one by giving table a list of `(key . value)` pairs.

```lisp
> (set k (table '((a . b) (c . d))))
(lit tab (a . b) (c . d))
```

The `vir` for tabs means we can use tables as virtual functions on
keys.

```lisp
> (k 'a)
b
> (k 'z)
nil
> (map k '(a c))
(b d)
```

Because we define a loc whose test is `isa!tab` and which expands 
into a `tabloc` expression, we can set table entries and also create 
new ones implicitly.

```lisp
> (set k!a 1 k!z 2)
2
> k
(lit tab (z . 2) (a . 1) (c . d))
```

Finally, `tabrem` removes all entries with certain keys from a table.

```lisp
> (tabrem k 'z)
((a . 1) (c . d))
> k
(lit tab (a . 1) (c . d))
```

The last code in the Bel source is for defining objects with named 
fields. First we define templates describing these objects, which 
are stored in a global list. The templates can then be used to define 
instances.

We call `tem` with the name of the template followed by pairs of field 
names and expressions to generate default values for those fields. 
E.g. this defines a two-dimensional point:

```lisp
(tem point x 0 y 0)
```

Although we used 0s here, you can use any code in a default 
expression. It will be stored in a closure and called each time a
default value is needed.

Having defined `point`, we can make new instances with make

```lisp
> (set p (make point))
(lit tab (x . 0) (y . 0))
```

We can supply initial arguments if we want, and if we do they 
override the defaults.

```lisp
> (set q (make point x 1 y 5))
(lit tab (x . 1) (y . 5))
```

Notice that nothing in a point says that it is a point. You can
easily change that by adding a field called e.g. type, but for 
maximum flexibility the language doesn't do it for you.

Since instances are tables, we can do to them everything we can do to 
tables. 

```lisp
> p!x
0
> (++ p!x)
1
> (swap p!x p!y)
1
> p
(lit tab (x . 0) (y . 1))
```

We can even add and remove fields, which is either powerful or
alarming depending on how you feel about such things.

We start to see the potential of intrasymbol characters and upon in 
functions like this one

```lisp
(set above (of > !y))
```

which tells if points are above another in the sense of having
greater y-coordinates.

```lisp
> (above q p (make point))
t
```

Finally, the function readas reads a table as an instance of a
template. The fields that we read are treated as if they were initial
arguments to make, which means if fields have been added to a 
template since what we're reading was written, it will automatically 
get the new fields with the default values. In my experience that's a 
convenient feature to have.

---

Thanks to Kartik Agaram, Trevor Blackwell, Patrick Collison, Daniel 
Franke, Joel Franusic, Jared Friedman, Daniel Gackle, Michael Hartl, 
Matt Knox, Shriram Krishnamurthi, Kevin Lacker, Jessica Livingston, 
Bill Moorier, Olin Shivers, Nick Sivo, and especially Robert Morris 
for help with Bel.
