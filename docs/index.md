# SPIL - SimPle LIsp implementation written in Go

[github page](https://github.com/avoronkov/spil/)

SPIL is a small functional language with lisp-like syntax, static type checking and some other features like lazy-evaluations, tail-call optimization and memoization.

## Disclaimer

I started SPIL as my own experimental pet-project during the self-isolation in April 2020.
The idea was to investigate how long does it take to create a programming language from scratch using other programming language (Go).

So I began with a limited number of basic concepts: 3 primitive types (boolean, integer, string) and 1 complex type (list), very simple LISP-like syntax, then I added user-defined functions, high-order functions, lambdas etc and now I'm trying to manage with generic functions and static type checking.

Though it's not a production ready language I found it very interesting and fun for some tasks (like solving problems from [projecteuler.net](https://projecteuler.net/)).
Also developing new programming language is very exciting by itself.
So maybe some day I will implement more reliable interpreter with fast runtime.

## Installation

```
$ go get github.com/avoronkov/spil
$ spil
(print "hello world!")
^D
hello world!
```

## Language overview

Well, it's a kind of Lisp, so you write you code with the constructions like that:
```Lisp
(print (- (* 2 3) 1))
(print (+ 1 2 3 4 5))
(print "this is true:" 'T "and this is false:" 'F)
(print "this is a raw list:" '(1 2 3 print hello))
```

Also it's (almost) pure functional language, so you have no mutable variables and no loops.
(Actually `print` is the only statement with side effects).

### Comments

Lines started with `;` or `#` are comments.

### Data types

SPIL (like most of other Lisps) has atoms and lists as basic data type.
Atoms include the following:

- Integers: `0` `1` `25` `1235` `-128` ...

- Booleans: `'T` `'F`

- Strings: `"hello world!"` `"foo"` `"bar"` ...

- Identifiers: `foo` `bar` `func` `if` ...

Lists include:

- unquoted s-expressions: `(foo 1 2 3)` `(print "hello")` ...

- and quoted (raw) ones: `'(this is "not" evaluated)` ...

### Everything is an expression

Every statement in SPIL is an expression i.e. every statement is r-value which can be returned from function or assigned to "variable".

### Basic functions

SPIL has the following builtin functions implemented in Go:

- `print` - prints values of expressions on stdout.

- Arithmetic operations: `+`, `-`, `*`, `/`, `mod`, `<`, `>`, `<=`, `>=`.

- Equality operator: `=`

- Functions to work with lists: `head`, `tail`, `append`, `list`, `empty`.

### User-defined functions

You may define you own function with keyword `def` (of `func`):
```Lisp
; (def <function-name> <function-parameters> <body-statement1> ...)
(def plus-one (n) (+ n 1))

(print (plus-one 3))
; 4
```

Return value of function is a return value of last expression in function.

Note that `def` defined so-called "pure" function i.e. its return value can depend only on its arguments.

Function may have multiple definitions with different set of arguments:
```Lisp
(def factorial (0) 1)
(def factorial (n) (* n (factorial (- n 1))))
```

### Control flows

SPIL has conditional operator `if` which has the following syntax:
```Lisp
(if  some-condition  return-value-if-true return-value-if-false)
```

Note that `if` is also an expression i.e. it has a return value.

### Recursion

SPIL has no loops. Instead it uses recursion as in example above:
```Lisp
(def factorial (0) 1)
(def factorial (n) (* n (factorial (- n 1))))
```

Note that such recursion is not very effective because it consumes call-stack.
That's why it's better to use tail-call recursion like that:
```Lisp
(def factorial (n) 1)
(def factorial (0 result) result)
(def factorial (n result) (factorial (- n 1) (* result n)))
```
SPIL has Tail Call Optimization so the result will be returned from `(factorial 0 result)` directly to the caller.

If you are not familiar with recursion and tail calls you may read a great book for functional programming beginners [Learn you some Erlang for great good](https://learnyousomeerlang.com/).

### Passing functions as arguments to other functions

You may pass function as an argument by its name:
```Lisp
(func plus-one (n) (+ n 1))

(func apply-func-to-ten (fn) (fn 10))

(print (apply-func-to-ten plus-one))
; 11
```

### Lambdas

You can define lambda-functions with `lamda` keyword.
```Lisp
(func apply-func-to-ten (fn) (fn 10))

(print (apply-func-to-ten (lambda (+ _1 1))))
; 11
```
Lambdas are very similar to regular functions but they have some difference:

- Lambda can grab values of variable from the context where lambda is defined:
```Lisp
(func apply-func-to-ten (fn) (fn 10))

(set n 5)

(print (apply-func-to-ten (lambda (+ _1 n))))
; 15
```

- Lambdas are designed to be small so they use short syntax of accessing arguments:
`_1`, `_2`, `_3` ... for accessing positional arguments and `__args` for accessing whole list of arguments.

### Lazy lists

You can use keyword `gen` to define finite or infinite lazy lists.
For example lazy-list of positive integers can be defined like this:
```Lisp
(def inc (n) (+ n 1))

; infinite lazy list of integers: (1 2 3 4 ...)
(set ints (gen inc 0))
```

`gen` has the following syntax:
```Lisp
(gen <iterator-function> <initial-state>)
```
When somebody asks for `head` of lazy lists then `iterator-function` is called with value of previous state.
Iterator should return one of the following:

- Empty list `'()` to indicate that list has ended.

- List with one element `(list value)` which will be returned as next element (head) in lazy-list and will be passed to the next call of iterator.

- List of two elements `(list value new-state)`. `value` will be returned by `head`, `new-state` will be passed to the next call of iterator.

For example the infinite list of Fibonacci numbers:
```Lisp
(def next-fib (prev)
	(set a (head prev))
	(set b (head (tail prev)))
	(list b (list b (+ a b))))
(set fibs (gen next-fib '(1 1)))

(print (take 10 fibs))
; '(1 2 3 5 8 13 21 34 55 89)
```

### Using modules

You can `use` other modules in your program:
```Lisp
(use "some-module.lisp")

(function-from-some-module ...)
```

You can also use module `std` which contains some useful functions like:

- `map`, `filter`, `reduce` - high order functions to work with lists

- `take`, `take-while` - take some elements from the list

- `drop` - drop somw elements from the list

- `length` - evaluate length of the list

- `nth`, `first`, `second`, `third` - access some element in list

- `concat` - concatenate some lists

- `inc`, `dec` - increment, decrement integer value

- `lines`, `words` - split string into list of lines or list of words.

### Big math
You can use big integers instead of int64 in calculations by adding `(use bigmath)` statement and the beginning of the main module.

```Lisp
(use bigmath)

(print (* 10000000 10000000 10000000))
; 1000000000000000000000
```

### Memoization

You can tell the interpreter to remember function results by defining function with `def'` (or `func'`) keyword.
As a result if such function is called with the same set of arguments twice then its result will be calculated only once.
Second time it will return the stored result.

```
(def' x2 (n) (print "evaluating x2" n) (* n 2))

(print (x2 5))
(print (x2 6))
(print (x2 5))
; evaluating x2 5
; 10
; evaluating x2 6
; 12
; 10
```

### Work with files

You can work with files as lazy-strings (?).
Well, it means that you can open file and iterate over its content with `head` and `tail` methods.
It may seems kinda low-lever so I've implemented functions `lines` and `words` to split string into lines of words
and these functions are also lazy.

Note that you need to use `std` module to access these functions.

```
(use std)

(set' file (open "somefile.txt"))

(print (map words (lines files))
```

Also note that operator `set'` is used instead of simple `set`.
It means that file will be automatically closed when interpreter leaves the current function scope.

(Writing into files is not implemented yet.)

## Types

You can specify types of your function parameters and function's return value.
```Lisp
(def contains (value:any '()) :bool 'F)
(def contains (value:any lst:list) :bool
	(if (= (head lst) value)
		'T
		(contains value (tail lst))))

(print (contains 4 '(1 3 5 8)))
```

The following builtin type are available: `:int`, `:str`, `:bool`, `:list`, `:func`, `:any`.

### Static type checking

SPIL checks the correctness of types usage in "compile time", i.e. before actual execution of the the program.
You can specify option "--check" (or "-c") for syntax and type checking of the program.
E.g. when you misplace the arguments in previous example (`(print (contains '(1 3 5 8) 4))`) you will get the following error:
```
$ spil -c example.lisp
__main__: contains: no matching function implementation found for [{:list {S': {Int64: 1} {Int64: 3} {Int64: 5} {Int64: 8}}} {:int {Int64: 5}}]
```

### Type casting

Sometimes you need to cast expressions types. E.g. in the following example:
```Lisp
(def ascending? (l:list) :bool
     (if (<= (length l) 1)
       'T
       (if (> (first l) (second l))
         'F
         (ascending? (tail l)))))

(print (ascending? '(1 2 3 5 8)))
```
you will get the error:
```
ascending?: >: Expected all integer arguments, found {:any <nil>} at position 0
```
because `nth` returns `:any` but `>` expects `:int`.
So you can fix it with casting first and second elements to `:int`:
```Lisp
(def ascending? (l:list) :bool
     (if (<= (length l) 1)
       'T
       (if (> (do (first l) :int) (do (second l) :int))
         'F
         (ascending? (tail l)))))
```
I may look strange but actually it's rather simple. SPIL has the following forms of types casting:
```Lisp
; convert result of function to :int
(def get-int () :int (function-returning-any) :int)

; variable var has type :int now
(set var (function-returning-any) :int)

; convert return of do-block to :int
(do (function-returning-any) :int)
```

### User defined types

You may define your own type with `deftype` statement:
```Lisp
; (deftype new-type parent-type)
(deftype :my-type :any)
```
It may be helpful in some scenarios, i.e. if we want to implement simple "type-safe" set:
```Lisp
(deftype :set :list)

(def set-new () :set '() :set)
(def set-add (elem:any s:set) :set
	(if (contains elem s)
	  s
	  (do (append s elem) :set)))

;; This will cause typecheck error:
(set-add '(1 2 2 4 5) 6)

;; This is OK
(set s1 (set-add (set-new) 1))
(set s2 (set-add s1 2))
(set s3 (set-add s2 2))
(set s4 (set-add s3 3))

(print s4 (length s4))
```
Note that you cannot use :list variable where :set is required, but you can pass :set anywhere where its parent type (:list) is accepted.

## Generic types (work in progress)

### Types with parameters

At the some moment I realized that it would be nice to restrict somehow types of list's elements and I added types parametrization.
It looks like this:
```Lisp
(def sum-list ('()) :int 0)
(def sum-list (l:list[int]) :int
     (+ (head l) (sum-list (tail l))))

(set li '(1 2 3 4 5) :list[int])

(print (sum-list li))
; 15
```
In this example we declare that `sum-list` accepts list of integers (`:list[int]`).
We also convert our "raw" list `'(1 2 3 4 5)` into list of integers using type casting.
Note that passing raw list to `sum-list` leads to "compile" time error:

```Lisp
(print (sum-list '(1 2 3 4 5)))
; __main__: sum-list: no matching function implementation found for [{:list {S': {Int64: 1} {Int64: 2} {Int64: 3} {Int64: 4} {Str: "5"}}}]
```

You may still use type `:list` list in your code: now it's an alias for type `:list[any]`.

### Parametrized function types

It is possible to specify function signature in type of function like this:

```Lisp
(def apply-fn-to-ten (fn:func[int,int]) :int (fn 10))

(def x2 (x:int) :int (* x 2))

(print (apply-fn-to-ten x2))
; 20
```

Syntax of function type is the following:
```Lisp
:func[types-of-arguments...,type-of-return-value]
```
Since every function in SPIL has a return-value, there should be at least one parameter to type `:func`.

#### Function type limitations

- `:func[...]` can be concerted into `:func` and vice versa.

- Lambdas always have type `:func`.

- If function has several bodies with different signature, its type is computed as `:func` and parameters matching is not getting checked in "compile" time.

### Generic types

Sometimes it is useful to have generic functions and generic types (hello, Go!).
Something like this:
```Lisp
(def nth (n:int l:list[a]) :a ...)

(set l '(1 2 3 4 5) :list[int])

(print (type (nth 3 l)))
; :int
```

There is a number of pre-defined generic type-parameters (`:a`, `:b`, ... `:e`)
but you can define your own parameter with `contract` statement:
```
(contract :cmp)

(def sort (l:list[cmp]) :list[cmp] ...)
```

These are not real contracts like in [Go draft](https://go.googlesource.com/proposal/+/master/design/go2draft-contracts.md)
yet but maybe later I will add possibility to add restriction on type parameters.

#### Generic types limitations

- Static type checking does not work properly with generic types in all cases.
I am going to fix it somehow later (e.g. by implementing contracts (?)).

## Examples

- Print first 30 prime numbers:

```Lisp
; increment number
(def inc (n:int) :int (+ n 1)) 

; test if number is prime
(def prime? (1) 'F) 
(def prime? (n) (prime? n 2)) 
(def prime? (n i)
    (if (> (* i i) n)
      'T  
      (if (= (mod n i) 0)
        'F  
        (prime? n (inc i)))))

; lazy list of positive integers
(set ints (gen inc 0) :list[int]) 

; lazy filter function
(def filter-int (pred:func l:list) :list
    (set
      filt
      \(if (empty _1) 
        '() 
        (if (pred (head _1))
          (list (head _1) (tail _1))
          (self (tail _1)))))
    (gen filt l))

; non-lazy take function
(def take (n:int l:list[a]) :list[a] (take n l '()))
(def take (0     l:list[a] acc:list[a]) :list[a] acc)
(def take (n:int l:list[a] acc:list[a]) :list[a]
     (take (- n 1) (tail l) (append acc (head l))))

; and finally:
(print (take 20 (filter-int prime? ints)))

;; '(2 3 5 7 11 13 17 19 23 29 31 37 41 43 47 53 59 61 67 71)
```

You can also find some more examples of code [here](https://github.com/avoronkov/spil/tree/master/examples)


## TODO

- [+] do-statement support

- [+] multiple function definition with pattern matching

- [+] pass command line arguments to the command

- [+] lazy lists

- [+] apply

- [+] anonymous functions (?)

- [+] function "list"

- [+-] restricted type casting and strict mode.

- [+] "length" and "nth" optimization for static listst.

- [+] Separate pragma parsing and loading std-lib first.

- Functions overloading for user defined types

- "error" and "catch" functions for runtime errors

- (TBD) Forbidden matching (:delete or something)

- Type of variable is vanished when placed into list.
