---
title: "Notes (CTFP 08/31): Functoriality"
---

# 8 Functoriality

## 8.1 Bifunctors

**bifunctor**: A functor in 2 arguments, or a mapping from 2 categories to a
single category

```haskell
class Bifunctor f
  bimap :: (a -> c) -> (b -> d) -> f a b -> f c d
  bimap g h = first f . second g
  first :: (a -> c) -> f a b -> f c b
  first f = bimap f id
  second :: (b -> d) -> f a b -> f c d
  second f = bimap id f
```

where

```haskell
bimap id = id
first id = id
second id = id
```

## 8.2 Product and Coproduct Bifunctors

Product bifunctor for `(,)`:

```haskell
instance Bifunctor (,) where
  bimap f g (x, y) = (f x, g y)
```

Coproduct bifunctor for `Either`:

```haskell
instance Bifunctor Either where
  bimap f _ (Left x) = Left (f x)
  bimap _ g (Right y) = Right (g y)
```

## 8.3 Functorial Algebraic Data Types

The `Const` functor is like a morphism to a terminal object (there's probably a
more precise way to formulate this).

The identity functor:

```haskell
data Identity a = Identity a

instance Functor Identity where
  fmap f (Identity x) = Identity (f x)
```

The identity functor is the identity morphism in `Cat` (the category whose
objects are categories and whose morphisms are functors).


Composition of bifunctors:

```haskell
newtype BiComp bf fu gu a b = BiComp (bf (fu a) (gu b))

instance (Bifunctor bf, Functor f, Functor gu) =>
  Bifunctor (BiComp bf fu gu) where
    bimap f1 f2 (BiComp x) = BiComp $ (bimap (fmap f1) (fmap f2)) x
```

This is an interesting puzzle trying to make sense of all these different type
shapes. Let's look at the definition of bifunctor again:

```haskell
class Bifunctor f where
  bimap :: (a -> c) -> (b -> d) -> f a b -> f c d
```

Remember that `f` has kind `TYPE -> TYPE -> TYPE`, specializes to
`(Bicomp bf fu gu)` in `Bifunctor (BiComp bf fu gu)`. More specializing:

```haskell
bimap :: (a -> c) -> (b -> d) -> BiComp bf fu gu a b -> BiComp bf fu gu c d
bimap f1 f2 (BiComp x) = BiComp $ (bimap (fmap f1) (fmap f2)) x
```


And since the `x` here is just the `bf (fu a) (gu b)` from the `BiComp`
definition, the inner bimap specializes to the `bf`'s Bifunctor instance:

```
(fu a -> fu c) -> (gu b -> gu d) -> bf (fu a) (gu a) -> bf (fu c) (gu d)
(fmap f1)         (fmap f2)        x
```


## 8.5 The Writer Functor

For some Kleisli category with objects `t a`:

```haskell
(>=>) :: (a -> t b) -> (b -> t c) -> (a -> t c)
return :: a -> t a
```

For some function `f :: a -> b`, we can build an `f' :: a -> t b` with `return`:

```haskell
(\x -> return $ f x) :: a -> t b
```

And then the Functor instance of `t` is:

```haskell
(>=>) id (\x -> return $ f x) :: t a -> t b
```

This works, because `id :: a -> a` specializes to `id :: t b -> t b`

```haskell
id :: t b -> t b
(\x -> return $ f x) :: b -> t c

(>=>) :: (a -> t b) -> (b -> t c) -> (a -> t c)
-- a ~ t b
(>=>) :: (t b -> t b) -> (b -> t c) -> (t b -> t c)
```

## 8.6 Covariant and Contravariant Functors

The Reader functor:

```haskell
type Reader r a = r -> a

instance Functor (Reader r) where
  fmap f g = f . g
```

The opposite of the Reader type:

```haskell
type Op r a = a -> r
```

Is there a Functor instance for `Op`? We would need to define

```haskell
fmap :: (a -> b) -> Op r a -> Op r b
fmap f ar = undefined

f :: a -> b
ar :: a -> r
```

Looks like we can't make a nontrivial `b -> r` from `a -> b` and `a -> r`, but
we could do it if our function was `b -> a`:

```haskell
contramap :: (b -> a) -> Op r a -> Op r b
contramap f ar = ar . f
```

So even though there's no functor instance for `Op`, there is an "opposite
functor" instance, which is called the "contravariant functor"

**contravariant functor**: a functor that reverses the direction of the
arrows in the mapped category

```haskell
class Contravariant f where
  contramap :: (a -> b) -> f b -> f a
```

The way I think about the difference between covariant and contravariant
functors is that covariant functors are "producers" and contravariant functors
are consumers.

A Producer of type `a` is something that can give type `a` output.
A Consumer of type `a` is something that can take type `a` input.

An example (or "instance") of an `a` Producer is `Maybe a`:

```haskell
data Maybe a = Just a | Nothing
```

This can produce values of type `a` in the `Just` constructor. There's even a
partial function `fromJust :: Maybe a -> a`.

An example of a Consumer of type `a` would be a test (or "predicate") on type
`a`:

```haskell
data Predicate a = Predicate { runPred :: a -> Bool }
```

An example of a predicate on `Integer` is:

```haskell
isEven :: Integer -> Bool
isEven n = if n % 2 == 0 then True else False
```

Now let's imagine we have a function `f :: a -> b`. How can we use `f` to
transform a Producer?

Well, if we can transform `a`'s into `b`'s, and our `p a` outputs `a`'s, then
if we feed our `a`'s output's into `f` we get `b`'s, which makes the whole
ensemble a producer of `b`'s.

We can express this pattern with the following typeclass definition:

```haskell
class Producer p where
  pmap :: (a -> b) -> p a -> p b
```

Now lets think about how an `f :: a -> b` can transform a Consumer.

If we have `c b` that consumes type `b` inputs, then we can wire up our `f`
function to the "input port" of our `c b` machine. This means that we can now
feed values of type `a` into the `c b` consumer, because our `f` will transform
them into values of type `b` first.

Concretely, imagine that have our `isEven :: Integer -> Bool` predicate and we
have a function `length :: [a] -> Integer` that gives the integer length of a
list. We can combine these two functions into a predicate:

```haskell
isLengthEven :: [a] -> Bool
isLengthEven = isEven . length
```

We can express this pattern with the following typeclass definition:

```haskell
class Consumer c where
  cmap :: (a -> b) -> c b -> c a
```

Producers are covariant functors, Consumers are contravariant functors.

## 8.7 Profunctors

Can something be Producer and a Consumer at the same time? Yes it can!

A simple example is a Profunctor, which is a consumer in the first type argument
and a producer in the second:

```haskell
class Profunctor p where
  dimap :: (a -> b) -> (c -> d) -> p b c -> p a d
  dimap f g = lmap f . rmap g
  lmap :: (a -> b) -> p b c -> p a c
  lmap f = dimap f id
  rmap (b -> c) -> p a b -> p a c
  rmap = dimap id
```

Think of `p a b` as like a pipeline that consumes `a`'s and produces `b`'s.

Functions are instances of profunctors:

```haskell
instance Profunctor (->) where
  dimap ab cd bc = cd . bc . ab
  lmap = flip (.)
  rmap = (.)
```

## 8.8 The Hom-Functor

Why is the hom-set `C(a,b)` a functor `C^OP × C -> Set`?

The objects in `C^OP` and `C` are the same, but the morphism directions are
reversed.

The hom-set maps pairs of objects (the start and end
of the morphism) to the set of morphisms between them.

Is this a functor?

- Every pair of objects `(a, b)` maps to a (possibly empty) set, so each `X` does
  have an `F(X)`
- A morphism in `C^OP × C` that transforms a pair `(a,b)` into a pair `(a',b')`
  is the pair of morphisms from `C`: `(f :: a' -> a`, `g :: b -> b')`.

This does map an `(a -> b)` to an `(a' -> b')` because:

```
(f . (h :: a -> b) . g) :: a' -> b'
```

## 8.9 Challenges

### 1

```haskell
class Bifunctor f
  bimap :: (a -> c) -> (b -> d) -> f a b -> f c d
  bimap g h = first f . second g
  first :: (a -> c) -> f a b -> f c b
  first f = bimap f id
  second :: (b -> d) -> f a b -> f c d
  second f = bimap id f

data Pair a b = Pair a b

instance Bifunctor Pair where
  bimap g f (Pair a b) = Pair (g a) (f b)
  first f = bimap f id
  second f = bimap id f
```

### 2

```haskell
f :: Maybe' a -> (Either (Const () a) (Identity a))
f Nothing = Left $ Const ()
f (Just x) = Right $ Just x

f' :: (Either (Const () a) (Identity a)) -> Maybe' a
f' (Left $ Const ()) = Nothing
f' (Right $ Just x ) = Just x
```

### 3

```haskell

data PreList a b = Nil | Cons a b

instance Bifunctor Prelist where
  bimap g f Nil = Nil
  bimap g f (Cons a b) = Cons (g a) (f b)
```

### 4

```haskell
data K2 c a b = K2 c

instance Bifunctor (K2 c) where
  bimap g f (K2 c) = K2 c

data Fst a b = Fst a

instance Bifunctor Fst where
  bimap g f (Fst a) = Fst (g a)

data Snd a b = Snd b

instance Bifunctor Snd where
  bimap g f (Snd b) = Snd (f b)
```


