---
title: "Notes (CTFP 01/31): Category"
---

# 1 Category: The Essence of Composition

**category**: A structure consisting of objects and arrows between objects,
such that arrows compose. That is, if a category has objects `A`, `B` and `C`
along with arrows `A -> B`, `B -> C`, then it also has the arrow `A -> C`.


## 1.1 Arrows as Functions.

Arrows are also called morphisms, which comes from the Greek "morphe" meaning
form.

Notation:

- `.` is composition, `(g . f)(x) = g(f(x))`
- `::` is type of, like `f :: Integer`
- `->` is an arrow, `f :: A -> B` means `f` is a morphism from `A` to `B`


## 1.2 Properties of Composition

1. Composition of arrows is associative, so that

    ```
    h . (g . f) = (h . g) . f = h . g . f
    ```

    This makes sense with the definition of a category. Suppose a category contains
    objects `A, B, C, D` and arrows:

    ```
    f :: A -> B
    g :: B -> C
    h :: C -> D
    ```

    Then by definition it contains:

    ```
    (f . g) :: A -> C
    (h . g) :: B -> D
    (h . g . f) :: A -> D
    ```

    Associativity just means that it doesn't matter what order we compose arrows:

    ```
    h . (g . f) == (h . g) . f == h . g . f
    ```


2.  There is an identity arrow `A -> A` for every object `A`, such that

    ```
    f :: A -> B
    g :: B -> A
    id_a :: A -> A

    f . id_a = f
    id_a . g = g
    ```


One thing that confuses me is that it seems like this implies that in some
cases that there can't be multiple arrows from one object to another.

Suppose a category `Cat` has objects `A`, `B` and arrows

```
f' :: A -> B
f  :: A -> B
g  :: B -> A
id_a :: A -> A
id_b :: B -> B
```

If `Cat` is a category, then the compositions of `g . f` and `g . f'` must
be arrows in the category:

```
g . f :: A -> A
g . f' :: A -> A
```

But the only arrow of type `A -> A` is `id_a`, so

```
g . f = g . f' = id_a
```

And equivalently:

```
f . g = f' . g = id_b
```

Which implies:


```
f' = f' . id_a = f' . g . f = id_b . f = f
```

So, either `Cat` is not a category, or `f' == f.`

cf. [Haskell Wikibook page on Category Theory](https://en.wikibooks.org/wiki/Haskell/Category_theory#Hask,_the_Haskell_category)

I think the important thing to keep in mind here is that when looking
at a possible structure to see if its a category, the structure is constant. There
aren't any implicit elements of `Cat` that could also satisfy the property
that e.g `(g . f)` is in `Cat`. We actually have to look in `Cat` to find if
something fits the properties. If not, it's not a category.

When I first approached this topic I confused the statements:

- For any arrows `g` and `f` in a category `C`, their composition `(g . f)` must
also be an arrow in `C`. (**Right**)
- If arrows `g` and `f` are arrows in `C`, add a new arrow `(g . f)` to `C`. (**Wrong**)

This really confused me for a bit, until I realized that when I was doing the
latter I was just trying to build a superset of `C` that actually was a
category. This is apparently called a "free category".

## 1. 4 Challenges

1. `I = (\x. x)` in the lambda calculus.
2. `compose = (\x y z. x (y z))`

3.  `compose I F X = (\x y z. x (y z)) I F X = I (F X) = F X`
    `compose F I X = (\x y z. x (y z)) F I X = F (I X) = F X`

4. No, links are not composable.
5. Friendships are not composable.
6. When the edges are composable, and every node has a self-directed edge.

----
