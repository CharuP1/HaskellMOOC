## Computing with lists

- There are two approaches to working with lists:

  - Write functions to do what you want, using recursive definitions that traverse the list structure.
  - Write combinations of the standard list processing functions.

- The second approach is preferred, but the standard list processing functions do need to be defined, and those definitions use the first approach (recursive definitions).
- We’ll cover both methods.


## Recursion on lists

- A list is built from the empty list $$[]$$ and the function $$cons\; :: \; a\rightarrow [a] \rightarrow [a] $$. The function $$cons$$ is usually written as the operator $$(:)$$ , in other words _:_ is the same as _`cons`_.
- Every list must be either
  - $$[]$$ or
  - $$(x : xs)$$ for some $$x$$ (the head of the list) and $$xs$$ (the tail).
  - $$(x :xs)$$ is an alternative syntax for $$cons\,x\, xs$$
- The recursive definition follows the structure of the data:
  - Base case of the recursion is $$[]$$.
  - Recursion (or induction) case is $$(x : xs)$$.

### Some examples of recursion on lists

#### Recursive definition of _length_

The length of a list can be computed recursively as follows:

~~~haskell
length :: [a] -> Int
length [] = 0                  # the base case
length (x:xs) = 1 + length xs  # recursion case
~~~

#### Recursive definition of _filter_

- _filter_ is given a _predicate_ (a function that gives a Boolean result) and a list, and returns a list of the elements that satisfy the predicate.

~~~haskell
filter :: (a->Bool) -> [a] -> [a]
Filtering is useful for the “generate and test” programming paradigm.
filter (<5) [3,9,2,12,6,4] -- > [3,2,4]

The recursive definition is straightforward:

filter :: (a -> Bool) -> [a] -> [a]
filter pred []    = []
filter pred (x:xs)
  | pred x         = x : filter pred xs
  | otherwise      = filter pred xs
~~~

## Computations over lists

- Many computations that would be for/while loops in an imperative language are naturally expressed as list computations in a functional language.
- There are some common cases:

  - Perform a computation on each element of a list: $$map$$
  - Iterate over a list, from left to right: $$foldl$$
  - Iterate over a list, from right to left: $$foldr$$

- It’s good practice to use these three functions when applicable
- And there are some related functions that we’ll see later


### Function composition

- We can express a large computation by “chaining together” a sequence of functions that perform smaller computations

1. Start with an argument of type $$a$$
2. Apply a function $$g :: a->b$$ to it, getting an intermediate result of type $$b$$
3. Then apply a function $$f :: b->c$$ to the intermediate result, getting the final result of type $$c$$

- The entire computation (first $$g$$, then $$f$$) is written as $$f \circ g$$.
- This is traditional mathematical notation; just remember that in $$f \circ g$$, the functions are used in right to left order.
- Haskell uses `.` as the function composition operator

  ~~~haskell
  (.) :: (b->c) -> (a->b) -> a -> c
  (f . g) x = f (g x)
  ~~~

## Performing an operation on every element of a list: _map_

- _map_ applies a function to every element of a list

  ~~~haskell
  map f [x0,x1,x2] -- > [f x0, f x1, f x2]
  ~~~

### Composition of maps

- _map_ is one of the most commonly used tools in your functional toolkit
- A common style is to define a set of simple computations using map, and to compose them.

  ~~~haskell
  map f (map g xs) = map (f . g) xs
  ~~~

This theorem is frequently used, in both directions.

### Recursive definition of _map_

~~~haskell
map :: (a -> b) -> [a] -> [b]
map _ []     = []
map f (x:xs) = f x : map f xs
~~~

## Folding a list (reduction)

- An iteration over a list to produce a singleton value is called a _fold_
- By convention, we will have a list with elements of type $$b$$, and the singleton value will have type $$a$$
- There are several variations: folding from the left, folding from the right, several variations having to do with “initialisation”, and some more advanced variations.
- Folds may look tricky at first, but they are extremely powerful, and they are used a lot! And they aren’t actually very complicated.


### Left fold: _foldl_

- foldl is _fold from the left_
- Think of it as an iteration across a list, going left to right.
- A typical application is $$foldl\, f\, a\, xs$$
- The $$xs :: [b]$$ argument is a list
- A useful intuition: think of the $$a :: a$$ argument is an “accumulator”.
- The function takes the current value of the accumulator and a list element, and gives the new value of the accumulator.

  ~~~haskell
  foldl :: (a->b->a) -> a -> [b] -> a
  ~~~

### Examples of _foldl_ with function notation

$$ \begin{aligned}
\mathtt{foldl\,f\,a\,[]} &amp; \rightsquigarrow &amp; a\\
\mathtt{foldl\,f\,a\,[x0]} &amp; \rightsquigarrow  &amp; f\,a\,x0\\
\mathtt{foldl\,f\,a\,[x0,x1]} &amp; \rightsquigarrow  &amp; f\,(f\,a\,x0)\,x1\\
\mathtt{foldl\,f\,a\,[x0,x1,x2]} &amp; \rightsquigarrow  &amp; f\,(f\,(f\,a\,x0)\,x1)\, x2\end{aligned} $$


### Examples of _foldl_ with infix notation

In this example, + denotes an arbitrary operator for f; it isn’t supposed to mean specifically addition.

~~~haskell
foldl (+) a []          -- > a
foldl (+) a [x0]        -- > a + x0
foldl (+) a [x0,x1]     -- > (a + x0) + x1
foldl (+) a [x0,x1,x2]  -- > ((a + x0) + x1) + x2
~~~

### Recursive definition of _foldl_

~~~haskell
foldl        :: (b -> a -> b) -> b -> [a] -> b
foldl f z0 xs0 = lgo z0 xs0
             where
                lgo z []     =  z
                lgo z (x:xs) = lgo (f z x) xs
~~~

### Right fold: _foldr_

- Similar to $$foldl$$, but it works from right to left

~~~haskell
foldr :: (a -> b -> b) -> b -> [a] -> b
~~~

### Examples of _foldr_ with function notation

$$ \begin{aligned}
\mathtt{foldr\,f\, a\, []            } &amp; \rightsquigarrow &amp; a\\
\mathtt{foldr\, f\, a\, [x0]          } &amp; \rightsquigarrow &amp; f\, x0\, a\\
\mathtt{foldr\, f\, a\, [x0,x1]       } &amp; \rightsquigarrow &amp;  f\, x0\, (f\, x1\, a)\\
\mathtt{foldr\, f\, a\, [x0,x1,x2]   } &amp; \rightsquigarrow &amp; f\, x0\, (f\, x1\, (f\, x2\, a))\end{aligned} $$

### Examples of _foldr_ with operator notation

~~~haskell
foldr (+) a []          -- > a
foldr (+) a [x0]        -- > x0 + a
foldr (+) a [x0,x1]     -- > x0 + (x1 + a)
foldr (+) a [x0,x1,x2]  -- > x0 + (x1 + (x2 + a))

### Recursive definition of _foldr_

~~~haskell
foldr            :: (a -> b -> b) -> b -> [a] -> b
foldr k z = go
          where
            go []     = z
            go (y:ys) = y `k` go ys
~~~

### Relationship between _foldr_ and list structure

We have seen that a list `[x0,x1,x2]` can also be written as

~~~haskell
    x0 :  x1 : x2 : []
~~~

Folding $$cons$$ (:) over a list using the empty list [] as accumulator gives:

~~~haskell
foldr (:)  [] [x0,x1,x2]
  -- >
  x0 :  x1 : x2 : []
~~~

This is identical to constructing the list using (:) and [] ! We can formalise this relationship as follows:

  $$foldr \; cons \; [] \; xs \; = \; xs$$

### Some applications of folds

~~~haskell
sum xs = foldr (+) 0 xs
product xs = foldr (*) 1 xs
~~~

We can actually “factor out” the $$xs$$ that appears at the right of each side of the equation, and write:

~~~haskell
sum      = foldr (+) 0
product  = foldr (*) 1
~~~

(This is sometimes called “point free” style because you’re programming solely with the functions; the data isn’t mentioned directly.)
