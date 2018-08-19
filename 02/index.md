---
title: "Notes (CTFP 02/31): Types and Functions"
---
# 2 Types and Functions

## 2.3 What are Types

**Set**: A category whose objects are sets and whose arrows are functions.

**Hask**: Set, except every set is extended with the bottom element `_|_`,
indicating non-termination.

The question of whether **Hask** is actually a category appears to be
an [interesting one](http://math.andrej.com/2016/08/06/hask-is-not-a-category/comment-page-1/)

[TODO: Workthrough of [Fast and Loose Reasoning is Morally Correct](https://www.cs.ox.ac.uk/jeremy.gibbons/publications/fast+loose.pdf)]

## 2.4 Why Do we need a Mathematical Model

**operational semantics**: Describing how a program runs by trying to
directly reason about how it operates on some idealized abstract machine.

**denotational semantics**: Describing how a program runs by building a
mathematical structure that corresponds to the program and proving theorems
about that structure.

## 2.5 Pure and Dirty Functions

**pure function**: A function that returns the same output for the same
input, and whose output and input are explicit.

I wonder if a function in, for example, `C` could be considered pure, but only
if we treat the whole state of the machine, or maybe the whole state of the
universe as the input and output types.

## 2.6 Examples of Types

**Void**: The type with no elements (other than `_|_`, of course, if we're in Hask)

**()**: Unit, the type with only one element: `()`


## 2.6 Challenges

1 through 4: [TODO: Memoization in Haskell.]



5.  4 functions:
    ```
    not :: Bool -> Bool
    id_bool :: Bool -> Bool
    (const True) :: Bool -> Bool
    (const False) :: Bool -> Bool
    ```

6.  ```
    absurdUnit :: Void -> ()
    absurdTrue :: Void -> Bool
    absurdFalse :: Void -> Bool

    id_Void :: Void -> Void
    id_unit :: () -> ()
    id_bool :: Bool -> Bool

    not :: Bool -> Bool
    (const True) :: Bool -> Bool
    (const False) :: Bool -> Bool

    (const True)  :: () -> Bool
    (const False) :: () -> Bool

    unit :: Bool -> ()
    ```

[TODO: Diagram]
