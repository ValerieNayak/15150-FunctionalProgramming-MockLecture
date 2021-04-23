# 15150-FunctionalProgramming-MockLecture
Lesson on continuation passing style (CPS) in 15-150, Functional Programming

Welcome to my lesson on continuation passing style (CPS)! I found this to be one of the most challenging parts of the course, so I'm excited to get to share this topic with others and hopefully clear up some of the confusions you're having now. Chances are I struggled with a lot of the same things!

## What is CPS?
Up till now, all the functions that we've worked with have been direct-style functions. A function `t1 -> t2` takes an argument of type `t1 ` and returns a value of type `t2`. Now instead with CPS, we want to apply our function to a value of type `t1` and a **continuation** of type `t2 -> ans`. So instead of directly returning a result of type `t2`, we will hand off our result to the continuation which will produce a result of type `ans`.

### CPS requires...
- the function takes in a **continuation**
- all calls to CPS functions (including itself) must be **tail calls**
- calls to continuations must be tail calls

Reminder: Tail recursive means no computations after the recursive call (i.e. - no casing on outcome, appending to lists, adding the result, etc). Additional computations you need to do can happen in your new continuation, which you'll often write as a lambda.

The main kinds of problems you'll see in CPS are usually **accumulation** and **control flow** problems.

## Examples
**Tip**: Often when you don't know how to approach a CPS problem, it helps to first code it using direct recursion, then think about how you can move computations into your continuation function and rewrite it in CPS.

### Factorial
#### Direct Recusion:
```
(* fact: int -> int *)
fun fact n = 
  if n = 0 then 1
  else n * fact (n-1)
```
Notice how this is not tail recursive, because after we get the result of our recursive call we multiply it by `n`.

#### CPS Version
```
(* fact_cps: int -> (int -> 'a) -> 'a *)
fun fact_cps n k =
  if n=0 then k 1
  else fact_cps (n-1) (fn res => k(n * res))
```
Observe how now, we've moved our additional computation into our continuation function.

### Evaluation trace of CPS sum over a list function
[Here are some great notes](http://www.cs.cmu.edu/~15150/resources/lectures/12/csum.pdf) created by Prof. Erdmann that walks through the same example we did, step by step. I'll also add the PDF to the GitHub for easy access.

### Tree Search: Uses success and failure continuations
Sometimes, you can have **more than one continuation function**. (Wild, right?) Sometimes you use **success and failure continuations** so you can have different behavior depending on if you "succeed" or "fail." A good example of this is search, where success means you found what you're looking for and failure means you didn't. Here's some code for tree search using CPS.
```
datatype 'a tree = EMPTY | Node of 'a tree * 'a * 'a tree
(* tsearch: ('a -> bool) -> 'a tree -> ('a -> b) -> (unit -> 'b) -> 'b *)
fun tsearch _ Empty _ fc = fc ()
  | tsearch p Node(l, x, r) sc fc =
    if p x then sc x
    else tsearch p l sc (fn () => search p r sc fc)
```