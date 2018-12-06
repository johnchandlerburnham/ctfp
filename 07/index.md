---
title: "Notes (CTFP 07/31): Functors"
---

# 7 Functor

**functor**: A functor is a mapping between categories.

```haskell
f :: a -> b

F f :: F a -> F b

F :: (a -> b) -> (F a -> F b)
```

Compositions of functions in the base category map to the composition of
the image of those functions:

```haskell
h = g . f
F h = F g . F f
```

An object's identity morphisms map to the identity morphisms of its image:

```haskell
F (id :: A -> A)  = id :: F A -> F A
```

**endofunctor**: A functor that maps a category to itself.

## 7.1 Functors in Programming

### 7.1.1 The Maybe Functor

```haskell
data Maybe a = Nothing | Just a
```

`fmap` lifts a function onto the functorial target category. For `Maybe`:

```haskell
fmap :: (a -> b) -> (Maybe a -> Maybe b)
```

In general:

```haskell
fmap :: Functor f => (a -> b) -> (f a -> f b)
```

The functor maps both objects and categories, but the objects are types
not values, so we need

```haskell
F :: * -> *
```

which is `Maybe.` See this [Stack Overflow question](https://stackoverflow.com/questions/33441140/why-is-pure-only-required-for-applicative-and-not-already-for-functor?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
for more detail.

### 7.1.2 Equational Reasoning

Equational reasoning is possible with pure functions, because the equality is
not hiding state mutation. Thus, reasoning about code in functional programming
language like Haskell becomes more explicitly mathematical.

### 7.1.3 Optional

Skipping `C++` content. Perhaps I will come back and do this with `Rust`

### 7.1.4 Typeclasses

Suppose `Maybe` is a Functor. `Maybe` is a type constructor. This means `Maybe`
itself is the mapping from objects to objects (`* -> *`) e.g. `Int -> Maybe Int.`

But to be a functor, `Maybe` needs a mapping from morphisms to morphisms. This
is `fmap`, which is declared in Haskell in the Functor class:

```haskell
class Functor f where
  fmap :: (a -> b) -> f a -> f b
```

To say that `Maybe` is an instance of this class, we have to give the
implementation of `fmap`:

```haskell
instance Functor Maybe where
  fmap f Nothing = Nothing
  fmap f (Just a) = Just (f a)
```

### 7.1.5 Functor in C++

Skipping `C++`.

### 7.1.6 The List Functor

```haskell
data List a = Nil | Cons a (List a)
```

An `fmap` for `List` has to have the type:

```haskell
fmap :: (a -> b) -> (List a -> List b)
```

If we have a `List` of `a`'s and a function that transforms `a`'s into `b`'s, what's the most natural way to get a `List` of `b`'s? To apply the transformation to each `a` in the list!

```haskell
instance Functor List where
  fmap _ Nil = Nil
  fmap f (Cons x t) = Cons (f x) (fmap f t)
```

### 7.1.7 The Reader Functor

In Haskell, a function type is constructed using the arrow type constructor:

```haskell
(->) a b
```

Because `(->)` is a type constructor that takes two arguments, `(->) a` is a
type constructor, like `List` or `Maybe`.

Let's give wrap type constructor `(->)` in a newtype called `Reader`:

```haskell
{-# LANGUAGE InstanceSigs #-}
newtype Reader r a = Reader { run :: r -> a }

instance Functor (Reader r) where
  fmap :: (a -> b) -> (Reader r a) -> (Reader r b)
  fmap f reader = Reader $ f . (run reader)
```

We could also say:

```haskell
fmap f (Reader g) = Reader $ f . g
```
## 7.2 Functors as Containers

The idea of a Functor as a container of some value or values is imperfect,
since sometimes a Functor will explicitly contain no values, as in the
case of the `Const` functor:

```haskell
{-# LANGUAGE InstanceSigs #-}
data Const c a = Const c

instance Functor (Const c) where
  fmap :: (a -> b) -> Const c a -> Const c b
  fmap _ (Const v) = (Const v)
```

A `Const c a` is a box containing a type `c` and a "phantom" type `a`. The
type `a` doesn't exist except in the type signature.

## 7.3 Functor Composition

Lets say we have a list of integers:

```haskell
ints :: [Int]
ints = [0,1,2,3,4,5,6,7,8,9]
```

Now suppose we want to apply the function:

```haskell
isPrime :: Int -> Bool
```
to this list. The specific implementation of `isPrime` isn't important,
it just returns `True` if the `Int` is prime and `False` if it is not
prime.

But we have a problem here, in that prime numbers are only defined for
integers greater than 1. `isPrime 3` should return `True` and `isPrime 4`
should return `False`, but `isPrime 1` is undefined and should return neither
of those things.

Now, you might be saying, why don't we just return `False` for any integer where
`isPrime` is undefined? This does seem like the simplest solution from
a programming perspective.

However, this solution could actually generate a lot of headache for us later on.

For example, let's say we wanted to write another function `isComposite`,
that returns `True` for integers with factors.

Mathematical intuition says that we should be able to do:

```haskell
isComposite :: Int -> Bool
isComposite = not . isPrime
```

But if we defined `isPrime 1` to be `False` then `isComposite 1` would return
`True`, which is wrong, because "composite" numbers are also only defined
for numbers greater than 1.

Now, suppose we have some code (maybe from a library) where `isPrime` and
`isComposite` do follow the rule that each is the logical inverse (the "not)
of the other.

But let's say that for whatever reason, the author decided to make `isPrime`
and `isComposite` partial functions, meaning that for any integers outside
their domain (less than or equal to 1) they return a runtime error.

Runtime errors, are annoying (and dangerous) so, let's say we want to fix their
code.

We could directly modify their code and write two new functions:

```haskell
safeIsPrime :: Int -> Maybe Bool
safeIsComposite :: Int -> Maybe Bool
```

So that `isPrime 1` return `Nothing` and `isComposite 4` returns `Just True`.

Editing other peoples' code is a lot of work though, and isn't always the
best solution. Is there a way we can turn make partial functions safe from
the "outside"?

There is! We know that

```haskell
safen :: Int -> Maybe Int
safen x
  | x > 1 = Just x
  | otherwise = Nothing

safeIsPrime x = isPrime <$> (safen x)
```

This `safen` function is essentially lifting our `Int` into a `Maybe Int` that's
constrained to only have the values that `isPrime` can safely be applied to.

## 7.4 Challenges

### 1

The functor laws are:

```haskell
fmap id = id
fmap (g . f) = (fmap g) . (fmap f)
```

The proposed definition `fmap _ _ = Nothing` breaks the identity law, because it
implies

```
fmap id (Just x) = Nothing /= id (Just x)
```

### 2

The functor for Reader is:

```haskell
newtype Reader r a = Reader { run :: r -> a }
fmap f (Reader g) = Reader $ f . g
```

`fmap id = id` because for all g `(id . g) = g`

`fmap (g . f) = (fmap g) . (fmap f)` because

```haskell
fmap (g . f) (Reader r) = Reader $ (f . g) . r
((fmap g) . (fmap f)) . (Reader r) = Reader $ f . (g . r)
```

function composition is associative, so

```haskell
(f . g) . r = f . (g . r)
```

### 3

Pass, I'm really in this for the Haskell.

### 4

The functor instance for List

```haskell
data List a = Nil | Cons a (List a)
instance Functor List where
  fmap _ Nil = Nil
  fmap f (Cons x t) = Cons (f x) (fmap f t)
```

By induction the first law `fmap id = id` works on a base case for `Nil`

```haskell
fmap id Nil = Nil = id Nil
```

and on the inductive step with `Cons`, that is,  assuming `fmap id t = id t`,

```haskell
fmap id (Cons x t) = Cons (id x) (fmap id t) = id (Cons x t)
```

The second functor law is the same

```haskell
fmap (g . f) Nil = Nil = ((fmap g) . (fmap f)) Nil
fmap (g . f) (Cons x t) = Cons ((g . f) x) (fmap (g . f) t) 
((fmap g) . (fmap f)) (Cons x t) = Cons ((g (f x)) ((fmap g) . (fmap f)) t)
```

Since the inductive step for `Cons` assumes

```haskell
fmap (g . f) t = ((fmap g) . (fmap f)) t
```

and `g (f x) = (g . f) x` by definition:

```haskell
fmap (g . f) (Cons x t) = ((fmap g) . (fmap f)) (Cons x t)
```haskell
