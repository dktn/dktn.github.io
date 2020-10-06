---
author: Adam Szlachta
title: Real World Functional Programming for Netguru
date: October 7, 2020
---

# About me

* currently - Restaumatic:
  * Haskell and PureScript
    * Ruby (on Rails), JavaScript, TypeScript
* past professional experience:
  * Elixir + Phoenix
  * Scala + Play
  * Java + Spring + Hibernate
  * C#
  * C++
  * PHP
  * Perl
* functional programming lab using Haskell - AGH

# Agenda

- a bit of history
- a bit of underlaying math
- functional vs purely functional programming
- functions, types and type classes
- algebraic data types
- side effects in purely functional programming
- functional programming in practice
- Haskell and PureScript on production
- questions/answers

# History

* 1936 - lambda calculus (Alonzo Church)
  * untyped, inconsistent
* 1940 - simply typed lambda calculus
  * consistent
* 1942 - category theory initial concepts (Eilenberg & Mac Lane)
* 1957 - Fortran (first programming language)
* 1958 - LISP (second programming language)
  * S-expressions
  * dialects: Common LISP, Scheme, Racket, Clojure
  * dynamically typed

# History (continued)

* 1972 - SASL (1976 procrastinating machine)
* 1978 - Milner (Hindley–Milner type system aka HM)
* 1985 - Miranda (SASL + algebraic types and HM)
* 1989 - Coq (interactive theorem prover)
* 1990 - Haskell
  * influenced Agda, Idris, Elm, PureScipt
  * influenced by Miranda
* 1999 - Agda (dependent types)
* 2007 - Idris (dependent types)

# Typing

## dynamic vs static

```javascript
var ala = "ma kota"; // JavaScript
```

```scala
var ala : String = "ma kota"; // Scala
```

## type inference

```scala
var ala = "ma kota"; // Scala
```

# Typing

## strong vs weak

```JavaScript
// JavaScript

4 + '7';   // '47'
4 * '7';   // 28
2 + true;  // 3
false - 3; // -3
```

```C
// C

void* p;
```

# Function type

```ruby
def inc(val)
  return val + 1
end
```

```haskell
inc :: Int -> Int
inc val = val + 1
```

```haskell
inc :: Int -> Int
inc = (+ 1)
```

# Underlaying math

- lambda calculus - computation model
  - early introduction of types
- logic (Curry-Howard correspondence)
- category theory

# Lambda calculus

`λx.x+1`

```
(α) λx.A    →α  λy.[y/x]A
(β) (λx.A)B →β  [B/x]A
(η) λx.Ax   →η  A         if x not free in A
```

# Simply typed lambda calculus

Introduces function type: `𝜎→𝜏`

`λx:Int.x+1`

`λf:Int→Int.f 3`


# Function types examples

```haskell
even :: Int -> Bool                           -- even 7                  → False
flip :: (a -> b -> c) -> b -> a -> c          -- flip (-) 4 6            → 2
filter :: (a -> Bool) -> [a] -> [a]           -- filter even [1,2,3,4]   → [2,4]
replicate :: Int -> a -> [a]                  -- replicate 3 7           → [7,7,7]
repeat :: a -> [a]                            -- repeat 8                → [8..]
zipWith :: (a -> b -> c) -> [a] -> [b] -> [c] -- zipWith (+) [1,2] [3,4] → [4,6]
zip :: [a] -> [b] -> [(a, b)]                 -- zip [1,2] [3,4]         → [(1,2),(3,4)]
(,) :: a -> b -> (a, b)                       -- (,) 1 2                 → (1,2)
map :: (a -> b) -> [a] -> [b]                 -- map  (*2) [1,2,3]       → [2,4,6]
fmap :: Functor f => (a -> b) -> f a -> f b   -- fmap (*2) [1,2,3]       → [2,4,6]
foldl :: Foldable t => (b -> a -> b) -> b -> t a -> b -- foldl (+) 0 [1,2,3] → 6
foldr :: Foldable t => (a -> b -> b) -> b -> t a -> b -- foldr (+) 0 [1,2,3] → 6
```

Hoogle - <https://hoogle.haskell.org>

# Polymorphism

* ad hoc (function overloading)
  * functions behave differently for different argument types
* parametric (C++ templates, Java generics)
* subtyping
* row polyhorphism etc.

```Java
// Java
List<String> list = new ArrayList<String>();
```

```haskell
-- Haskell
map  ::              (a -> b) -> [a] -> [b]  -- parametric
fmap :: Functor f => (a -> b) -> f a -> f b  -- ad hoc
-- for list:   (a -> b) -> [a] -> [b]
-- for tree:   (a -> b) -> Tree a -> Tree b
```

# Lambda cube

![Lambda cube](images/lambda-cube.png)


# Functional programming

* First class functions
* Unmatched degree of abstraction and composition
  * `const`, `id`, `($)`, `(.)`
* Function types (in typed languages)
* Declarative style (in contrast to imperative)
  * no `for`, `while` and `until` loops! (`fold`, `map` and recursion instead)
  * `if` is an expression (like `p ? a : b` "operator")

```haskell
const :: a -> b -> a
id    :: a -> a
($)   :: (a -> b) -> a -> b
(.)   :: (b -> c) -> (a -> b) -> a -> c
```

CH-correspondence

# Functional primitives examples

```haskell
const :: a -> b -> a
id    :: a -> a
($)   :: (a -> b) -> a -> b
(.)   :: (b -> c) -> (a -> b) -> a -> c
```

```haskell
> const 7 8
7
> id 9
9
> (*2) $ 7
14
> (*2) . (+2) $ 3
10
```

# Quicksort

```c
void qsort(int a[], int lo, int hi) {
  int h, l, p, t;
  if (lo < hi) {
    l = lo;
    h = hi;
    p = a[hi];
    do {
      while ((l < h) && (a[l] <= p))
          l = l+1;
      while ((h > l) && (a[h] >= p))
          h = h-1;
      if (l < h) {
          t = a[l];
          a[l] = a[h];
          a[h] = t;
      }
    } while (l < h);
    a[hi] = a[l];
    a[l] = p;
    qsort( a, lo, l-1 );
    qsort( a, l+1, hi );
  }
}
```

# Haskell code example

```haskell
qsort :: Ord a => [a] -> [a]
qsort []     = []
qsort (x:xs) =
  let smaller = filter  (< x) xs
      bigger  = filter (>= x) xs
  in qsort smaller ++ [x] ++ qsort bigger
```

* pattern matching
* mathematical, recursive style
  * nearly equivalent to definition

# Purely functional programming

* referential transparency
* side effects management (how?)

```ruby
def inc(val)
  puts "I'm incrementing"
  dropBombs(7)
  return val + 1             + rand(2)
end
```

```haskell
inc :: Int -> Int
inc val = val + 1            -- `inc 1` can be replaced by `2`
```

```haskell
inc :: Int -> IO Int
inc val = do
  putStrLn "I'm incrementing"
  pure $ val + 1
```

# Side effects

* Which side effects exactly can be performed
* What environment the function can be executed in

```haskell
cancelDeliveryJob ::
  ( DBQueryEff effs
  , DBUpdateEff effs
  , Member Time.L effs
  )
  => DeliveryJobId
  -> Eff effs (Either ValidationErrors ())
cancelDeliveryJob deliveryJobId = tryUpdate `catchError` onConstraintError
  where
  -- business logic here
```

# Side effects (continuation)

```haskell
generateCoupons ::
  ( Member Time.L effs
  , Member Random.L effs
  , DBQueryEff effs
  , DBUpdateEff effs
  , Member Restaurant.L effs
  , Member (Exc InternalError) effs
  )
  => Int
  -> Coupon
  -> Eff effs (Either ValidationErrors [Coupon])
generateCoupons count coupon = do
  -- business logic here
```

# Laziness

```haskell
take :: Int -> [a] -> [a]

> take 5 [1..]
[1,2,3,4]
```

* provides beautiful abstractions
* often overemphasized feature
* causes problems
  * complexity difficult to judge
  * difficult to "debug"
* some modern languages don't really promote it (PureScript, Idris)

# Type classes

```haskell
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool

class Eq a => Ord a where
  compare :: a -> a -> Ordering
  (<)     :: a -> a -> Bool
  (<=)    :: a -> a -> Bool
  (>)     :: a -> a -> Bool
  (>=)    :: a -> a -> Bool
```

```haskell
isPrefixOf :: Eq a => [a] -> [a] -> Bool
sort :: Ord a => [a] -> [a]

> isPrefixOf "Ala" "Ala ma kota"
True

> sort "Ala ma kota"
"  Aaaaklmot"
```

# Algebraic data types

Kind fo composite types. Two classes of algebraic types are:

* product types (i.e., tuples and records)
* sum types (tagged unions, disjoint unions, variants, coproduct)

```haskell
-- product type
data Time = Time Hour Minute Second

-- sum type
data Weekday = Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday

-- product, sum, recursion and parametrization
data List a = Nil | Cons a (List a)

data Tree a = Empty
            | Leaf a
            | Node Tree Tree
```
# Pattern matching

```haskell
isEmptyList :: [a] -> Bool
isEmptyList [] = True
isEmptyList _  = False

isEven :: Int -> Bool
isEven v | v < 0 = isEven (-v)
isEven 0         = True
isEven 1         = False
isEven v         = isEven (v - 2)
```

# Pattern matching (continuation)

```haskell
data Tree = Empty
          | Leaf Int
          | Node Tree Tree

tree = Node
        (Node
          (Leaf 3)
          (Leaf 8)
        )
        (Node
          (Leaf 7)
          (Leaf 9)
        )

longestBranch :: Tree -> Int
longestBranch Empty             = 0
longestBranch (Leaf _)          = 1
longestBranch (Node left right) = 1 + max (longestBranch left) (longestBranch right)
```

# Category theory abstractions

* Monoid (Semigroup + neutral element)
* Functor
* Applicative
* Monad

```haskell
(<>)  :: Semigrup a => a -> a -> a
> "Ala" <> " " <> "ma kota"
"Ala ma kota"
> [1,2] <> [3,4]
[1,2,3,4]
> Sum 2 <> Sum 3
Sum {getSum = 5}
> Product 2 <> Product 3
Product {getProduct = 6}
> Any False <> Any True <> Any False
Any {getAny = True}
> EQ <> GT <> LT
GT
```

# Category theory abstractions (continuation)

* Functor
* Applicative
* Monad

```haskell
(<$>) :: Functor f     =>   (a -> b) -> f a -> f b   -- function
(<$>) :: Applicative f => f (a -> b) -> f a -> f b   -- "packed" function
(=<<) :: Monad f       => (a -> f b) -> f a -> f b   -- Kleisli arrow
```

# Haskell and PureScript on production

* Huge amount of libraries
* Great community
* Lots of examples (stackoverflow etc.)
* Pretty good tooling
  * VSCode (Haskell Language Server)
  * stack
* Refactoring, deep changes are a bliss (almost)
  * Types will lead you
* Low maintainance

# Functional programming

* Extremely rewarding and satisfying
* May be challenging for beginners
* Still very recommended to beginners!
  * eyes opening
  * teaches good style
  * teaches thinking in terms of abstractions
  * helps understanding programming in general

# Sources

* Learning resources:
  * <http://learnyouahaskell.com>
  * <https://wiki.haskell.org/Typeclassopedia>
  * <https://www.slideshare.net/kenbot/your-data-structures-are-made-of-maths>
  * many others...

* Links
  * <https://www.haskell.org>
  * <https://hoogle.haskell.org> (lookup by type signature)
  * <https://hackage.haskell.org> (libraries)
  * <https://docs.haskellstack.org> (stack tool)

* For Ruby
  * <https://dry-rb.org> dry-monads and other "dry" gems

# Q&A

* adam.szlachta@gmail.com
* <https://www.linkedin.com/in/adamszlachta>
