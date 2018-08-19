---
title: "Notes (CTFP 06/31): Simple Algebraic Data Types"
---

# 6 Simple Algebraic Data Types

## 6.5 Challenges

1.  Isomorphism:

    ```
    f :: Maybe a -> Either () a
    f Nothing = Left ()
    f Just a = Right a

    g :: Either () a -> Maybe a
    g (Left ()) = Nothing
    g (Right a) = Just a
    ```

2.  Life is too short for OOP boilerplate. Break the cycle Morty, rise above,
    focus on FP.

5.  Let's think about all the possible values of `a + a` and `2 x a`:

    ```
    a + a ~ Either a a = Left a | Right a
    2 x a ~ (Bool, a)
    ```

    But `(Bool, a)` is just

    ```
    (True, a)
    (False, a)
    ```

    so we have the isomorphism:

    ```
    f :: Either a a -> (Bool, a)
    f (Left a) = (True, a)
    f (Right a) = (False, a)

    g :: (Bool, a) -> Either a a
    g (True, a) = Left a
    g (False, a) = Right a
    ```
