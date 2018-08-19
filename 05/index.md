---
title: "Notes (CTFP 05/31): Products and Coproducts"
author: jcb
date: 2017-12-15
tags: notes, haskell, math, category-theory, ctfp
---
# 5 Products and Coproducts

## 5.1 Initial Object

**initial object**: The initial object is the object that has one and
only one morphism going to any object in the category.

This is like `Void`.

## 5.2 Terminal Object

**terminal object**: The terminal object is the object with one and
only one morphism coming to it from any object in the category.

This is like `()`

**poset**: A partially ordered set.

## 5.3 Duality

**opposite category**: For any category `C`, the opposite or dual category
`C^op` is `C` with all the arrows reversed. If `f` and `g` are arrows
in `C`, then

```
g . f = f^(op) . g^(op)
```

## 5.4 Isomorphisms

**inverse**: If for two objects `A` and `B` there are
two arrows `f :: A -> B` and `g :: A -> B`, such that:

```
f . g = id_A
g . f = id_B
```

`f` and `g` are inverses, or isomorphisms, and `A` and `B` are said
to be isomorphic.


### Theorem: Any category has at most one initial object (up to isomorphism)
Proof:

Suppose a category `C` has two initial objects `I1` and `I2`. Then by
definition it has the identity arrows:

```
id_I1 :: I1 -> I1
id_I2 :: I2 -> I2
```

And because `I1` and `I2` are initial objects, there must be only one
arrow from `I1` to `I2`, and only one arrow from `I2` to `I1`:

```
f :: I1 -> I2
g :: I2 -> I1
```

But if `f` and `g` are arrows in `C` then so are

```
(f . g) :: I2 -> I2
(g . f) :: I1 -> 11
```

And since an initial object has only one arrow to every other object in
the category, including itself:

```
(f . g) = id_I2 :: I2 -> I2
(g . f) = id_I1 :: I1 -> 11
```

And therefore `f` and `g` are inverses, so `I1` and `I2` are isomorphic.

Furthermore, since `I1` and `I2` are initial objects, `f` and `g` are
the only arrows between them, and therefore are unique isomorphism.

## 5.5 Products

**product**: A product of two objects `X1` and `X2` in a category `C` is an
object `X` with two arrows

```
p1 :: X -> X1, p2 :: X -> X2
```

with the property
that for all objects `Y` in `C` with arrows

```
f1 :: Y -> X1, f2 :: Y -> X2
```

there is a unique morphism `f :: Y -> X` such that

```
f . p1 = f1, f . p2 = f2
```

One way to think about this is whenever a product of `X1` and `X2` exists,
all the objects that have arrows to `X1` and `X2` have arrows to the product
object `X`. In other words, `X` is the most "downwind" or terminal-like
object in the collection of objects that have morphisms to `X1` and `X2`


## 5.6 Coproduct

**coproduct**: The dual of the product. Take the definition of the product
and reverse the arrows. In other words, the coproduct `X` of `X1` and `X2`
is the object that has arrows to every object that both `X1` and `X2` have
arrows to. It is the most "upwind", or initial-like object in
the collection of objects that have arrows from both `X1` and `X2`.

## 5.8 Challenges

1.  See Theorem in 5.4

2.  Okay, so the arrows in a poset represent the less-than-or-equal operator
    `<=`, so for any object in the poset `A`, all objects `B` such that `A <= B`
    have an arrow from `B` to `A`.

    For the product of two objects `X` and `Y`, let's consider all the objects
    `Z_n` that have arrows to both `X` and `Y.`

    So any two objects in `Z_n`, `Z1, Z2` have arrows:

    ```
    Z1 -> X, Z1 -> Y
    Z2 -> X, Z2 -> Y
    ```

    Now since the poset is a partial order, we cannot assume that there
    are arrows between any objects in `Z_n`. (Nor can we assume there
    are arrows `X -> Y`, or `Y <- X`.) Because of this in some posetal
    categories, the product may be undefined for two objects, for example, in
    the category:

    ```
    Objects: X, Y, Z1, Z2

    Arrows:
    Z1 -> X, Z1 -> Y, Z1 -> Z1
    Z2 -> X, Z2 -> Y, Z2 -> Z2
    X -> X
    Y -> Y

    Diagram:

    Z1 -> X
    |     ^
    v     |
    Y <- Z2
    ```

    This satisfies the category properties of composition and identity (since
    arrows in the above category can only be composed with identity arrows) and
    the partial order properties, but there is no defined product of `Y` and
    `X`

    For the product of `X` and `Y` to exist, there must be some element `Z` of
    `Z_n` such that all objects `Z_i` in `Z_n` have arrows `Z_i -> Z`. In other
    words, if arrows in a posetal category represent "less-than-or-equal", the
    product of `X` and `Y` is the greatest of all objects less than or equal to
    `X` and `Y`, or the greatest lower bound of `X` and `Y`.

    Interestingly, if e.g. `X -> Y` then `X` is in `Z_n` because of `X -> X`.
    And by definition is the greatest lower bound (because all `Z_i` have `Z_i
    -> X`). So if there is an arrow between `X` and `Y`, the product is simply
    the minimum of the two.

3.  The coproduct of two objects `X` and `Y` in a posetal category is the least
    upper bound of `X` and `Y`, that is, the smalllest object that is greater
    than or equal to both. It's exactly like the product, but with the arrows
    reversed.

4.  I have no favorite languages other than Haskell.
5.  So despite the C++ code, I think I can translate this problem into something

    I can do:

    ```
    i :: Int -> Int
    i x = x

    j :: Bool -> Int
    j x = if x then 1 else 0
    ```

    `Either` in Haskell is:
    ```
    Either a b = Left a | Right b

    Left :: a -> Either a b
    Right :: b -> Either a b
    ```

    `Either Int Bool` is a "better" coproduct of `Int` and `Bool` than `Int` if
    there is a unique arrow `unleft` from `Either Int Bool` to `Int` such that:

    ```
    (unleft . Left) = id_Int
    ```

    This function exists:

    ```
    unleft (Left x) = x
    unleft _ = 0
    ```

    Note that while `(unleft . Left) = id_x`, `(Left . unleft) \= id_Either`,
    because `Left (unleft (Right True) = Left 0`. If both composition equalled
    identity, then the two types would be isomorphic, but they are not. Either
    Int Bool is epimorphic (surjective) on Int, and Int is monomorphic
    (injective) on Either Int Bool.

    One thing that confuses me though, is that it seems like `unleft` isn't
    unique, because we could replace whatever number gets produced from a
    `Right` with any `Int`... So I'm clearly missing something here.

    Oh I see! The morphism from `Either Int Bool -> Int` is unique *given* two
    injections from `Int -> Int` and `Bool -> Int`. So the morphism shouldn't
    be named `unleft` at all, but actually `embed`, because it embeds an Either
    in `Int` space.

    Let's look at our category:

    ```
    Objects:
    Int, Int, Bool, Either Int Bool

    Arrows:

    i :: Int -> Int
    i n = n

    j :: Bool -> Int
    j True = 1
    j False = 0

    Left :: Int -> Either Int Bool
    Left n = Left n

    Right :: Bool -> Either Int Bool
    Right b = Right b

    embed :: Either Int Bool -> Int
    embed (Left n) = i n
    embed (Right b) = j b

    ```

    `embed` is unique for any `i` and `j` because it's just applying `i` and `j`
    based on which side of the `Either` we're on.

    And that's how we get the `factorizer` from the text:

    ```
    factorizer :: (a -> c) -> (b -> c) -> Either a b -> c
    factorizer i j (Left a) = i a
    factorizer i j (Right b) = j b
```

6. `id_Int = embed . Left`, but `id_Either \= Left . embed`, because
   `(Left . embed) (Right True) = Left 1`

    So there is an `Either Int Bool -> Int` arrow that factorizes `Int -> Int
    but not an `Int -> Either Int Bool` that factorizes `Either Int Bool ->
    Either Int Bool`

    Another way to look at this is that if we try to treat `Int` as an `Either`,
    we don't know if any given integer is supposed to be a `Left` or a `Right`.

    Like, if `j` sends `True, False` to `1, 0` and `i` sends `1, 0` to `1, 0`
    we don't know if applying `m :: Int -> Either Int Bool` to `1` should
    be a `Left 1` or a `Right True`.

7.  Our Category:

    ```
    i :: Int -> Int
    i n
      | n < 0 = n
      | otherwise = n + 2

    j :: Bool -> Int
    j b = if b then 1 else 0

    Left :: Int -> Either Int Bool
    Right :: Bool -> Either Int Bool
    ```

    Suppose Int is a "better" coproduct of Int and Bool with injections `i` and
    `j` than `Either Int Bool`. Then there must be a unique morphism `m :: Int
    -> Either Int Bool` such that:

    ```
    m . i = Left
    m . j = Right
    ```

    If on the other hand there is another morphism `m' :: Int -> Either Int Bool`,
    `m /= m'` such that

    ```
    m' . i = Left
    m' . j = Right
    ```

    Then `m` would not be a unique factorizing morphism and therefore `Int`
    could not be a better coproduct of `Int` and `Bool` than `Either Int Bool`

    Here's a candidate for `m`:

    ```
    m :: Int -> Either Int Bool
    m n
     | n == 0 = Right False
     | n == 1 = Right True
     | n < 0 = Left n
     | otherwise = Left (n - 2)
    ```

    This looks pretty unique at first glance.

    However.

    We can show that there is a unique morphism `f = (factorize i j) ::
    Either Int Bool -> Int`:

    ```
    factorize :: (Int -> Int) -> (Bool -> Int) -> Either Int Bool -> Int
    factorize i j (Left n)  = i n
    factorize i j (Right b) = j b
    ```

    such that

    ```
    f . Left = i
    f . Right = j
    ```

    So if both `m` and `f` are unique morphisms, then

    ```
    f . Left = f . (m . i) = (f . m) . i = i
    f . Right = f . (m . j) = (f . m) . j = j
    => (f . m) = id_Int

    m . i = m . (f . Left) = (m . f) . Left = Left
    m . j = m . (f . Right) = (m . f) . Right = Right
    => (m . f) = id_EitherIntBool
    ```

    which implies that `m` and `f` are isomorphisms. And since `m` and `f`
    are unique morphisms that fit the above conditions, `m` and `f` are
    unique isomorphisms. Which means `Either Int Bool` and `Int` are uniquely
    isomorphic, and are equivalent up to unique isomorphism. In which case
    `Int` cannot be a *better* coproduct that `Either Int Bool`.

    On the other hand, if `m` is not unique, there must be another `m' :: Int
    -> Either Int Bool`, in which case `Int` does not satisfy the universal
    property of coproducts.

    Actually, I think this is a general argument which applies to any possible
    candidate coproduct. If we already know that there is a coproduct in the
    category that satisfies the universal property, then any other other
    coproduct that satisfies the property must be uniquely isomorphic to the
    first, since their isomorphisms uniquely factorize their injections.

    Wow. That's a mathy paragraph. I begin to see why math explanations
    sound as impenetrable as they usually do...

8.  `()` is an inferior coproduct of `Int` and `Bool` than `Either Int Bool`,
    because there is only one unique morphism `unit :: Either Int Bool -> ()`,
    but no morphisms from `() -> Either Int Bool` that factorize injections
    from `unit :: Int -> ()` and `unit :: Bool -> ()`.

    But the question is asking for an inferior candidate that admits multiple
    morphisms between it and Either Int Bool.

    `Int` is an inferior coproduct of Int and Bool. Suppose we have
    the morphisms:

    ```
    Left :: Int -> Either Int Bool
    Right :: Bool -> Either Int Bool

    p :: Int -> Int
    p = (+3)

    q :: Bool -> Int
    q True = 1
    q False = 0
    ```

    Is there a unique morphism

    ```
    m :: Int -> Either Int Bool
    ```

    such that

    ```
    m . p = Left
    m . q = Right
    ```

    No! Because `p` and `q` don't map any values to `2 :: Int`, we can have

    ```
    m :: Int -> Either Int Bool
    m 0 = Right False
    m 1 = Right True
    m 2 = (a :: Either Int Bool)
    m n = n - 3
    ```

    We can return any `Either Int Bool` for `m 2` and `m` will still
    factorize, so we actually have infinite morphisms that satisfy the
    property. Therefore, Int is a very bad coproduct for Either Int Bool.

    On the other hand, the fact that we can use `Int` as a coproduct at all
    (even an inferior one) is convenient, because it lets use model
    coproducts (and other GADTs) on computers that only know about
    discrete quantites.
