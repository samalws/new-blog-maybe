# Dedekind Cuts in Haskell, Part 1
Dedekind cuts are really elegant.
For those of you unfamiliar with Dedekind cuts, it's possible to describe any real number (i.e., irrational or rational) by just providing the set of rational numbers it's less than, and the set of rational numbers it's greater than.
For example, 1 expressed as a Dedekind cut would be `({n | 1 < n}, {n | 1 > n})`.
The square root 2 (an irrational number!) would be expressed as a Dedekind cut as `({n | n > 0 and 2 < n*n}, {n | n < 0 or 2 > n*n})`.
It turns out that all the operations you'd expect to work on numbers (+, *, ^, etc) can be defined on Dedekind cuts, too.
So in order to explore Dedekind cuts more thoroughly, let's try to implement them in Haskell...
## Definitions
First, let's write out the definition of a Dedekind cut, in Haskell:
```Haskell
data Dedekind = Dedekind { lt :: Rational -> Bool, gt :: Rational -> Bool }
```
This matches pretty closely with how I defined Dedekind cuts earlier, except instead of providing sets of rationals greater and less than the number, we instead give "less than" (`lt`) and "greater than" (`gt`) operators.
We'll see how this ends up easier to work with later.
Next, let's add some helper functions helping us relate Dedekinds to Rationals:
```Haskell
geq :: Dedekind -> Rational -> Bool
a `geq` b = not (a `lt` b)
leq :: Dedekind -> Rational -> Bool
a `leq` b = not (a `gt` b)
eq :: Dedekind -> Rational -> Bool
a `eq` b = (a `leq` b) && (a `geq` b)
neq :: Dedekind -> Rational -> Bool
a `neq` b = not (a `eq` b)
```

## Some Basic Dedekind Cuts
So now that we know what a Dedekind is, let's start making some.
First, we'll make rational Dedekinds:
```Haskell
rationalDedekind :: Rational -> Dedekind
rationalDedekind n = Dedekind (\m -> n < m) (\m -> n > m)
```
In our set notation from before, this means `rationalDedekind n = ({m | n < m}, {m | n > m})`.
Now let's make Dedekinds for square roots:
```Haskell
sqrtDedekind :: Rational -> Dedekind
sqrtDedekind n = Dedekind (\m -> m > 0 && n < (m*m)) (\m -> m < 0 || n > (m*m))
```
In our set notation from before, this means `sqrtDedekind n = ({m | m > 0 and n < m*m}, {m | m < 0 or n > m*m})`,
which can be verified to be the sets of numbers that n is less than and greater than, respectively.

## Some Operations
Now let's figure out some operations on Dedekind cuts.
The first, and most basic, operation is negation.
For this, we'll take advantage of the fact that `-n < m` if and only if `n > -m`, and `-n > m` if and only if `n < -m`:
```Haskell
neg :: Dedekind -> Dedekind
neg n = Dedekind (\m -> n `gt` (-m)) (\m -> n `lt` (-m))
```
In set notation, this means `neg n = ({m | n > -m}, {m | n < -m}) = ({m | -n < m}, {m | -n > m}) = -n`, which is what we wanted.

Next, let's figure out how to do addition of a Dedekind cut and a rational number
(it turns out addition of two Dedekind cuts is really hard, so we'll do that in a later post).
For this, we'll take advantage of the fact that `n + m < a` if and only if `n < a - m`, and `n + m > a` if and only if `n > a - m`:
```Haskell
add :: Dedekind -> Rational -> Dedekind
n `add` m = Dedekind (\a -> n `lt` (a - m)) (\a -> n `gt` (a - m))
```
In set notation, this means ```n `add` m = ({a | n < a - m}, {a | n > a - m}) = ({a | n + m < a}, {a | n + m > a}) = n + m```, which is what we wanted.

Using a similar method, we can define multiplication of a Dedekind cut and a rational number.
This time, we need to add some edge cases for 0 and negatives:
```Haskell
mul :: Dedekind -> Rational -> Dedekind
n `mul` m
  | m == 0 = rationalDedekind 0
  | m < 0  = (neg n) `mul` (-m) -- n*m = (-n)*(-m)
  | otherwise = Dedekind (\a -> n `lt` (a / m)) (\a -> n `gt` (a / m)) -- ({a | n < a/m}, {a | n > a/m}) = ({a | n*m < a}, {a | n*m > a}) = n*m
```

To wrap up this section, let's define subtraction and division by piggybacking off of our definitions above:
```Haskell
sub :: Dedekind -> Rational -> Dedekind
n `sub` m = n `add` (-m)
divide :: Dedekind -> Rational -> Dedekind
n `divide` m = n `mul` (1/m)
```

## Ceiling and Floor
Now that we have some basic Dedekind cuts and operations on them, let's figure out how to read their value once we're done manipulating them.
Eventually, we'll create a way to print out numbers to a certain amount of decimal precision, but for now this should be enough to figure out the approximate values of numbers.

Let's do floor first. For this, we'll take advantage of the following rules:
- If `0 ≤ n < 1`, `flr n = 0`
- `flr (n+1) = (flr n) + 1`
- `flr (n-1) = (flr n) - 1`
Using these rules, we can create the algorithm:
```Haskell
flr :: Dedekind -> Int
flr n
  | n `geq` 0 && n `lt` 1 = 0
  | n `lt` 0 = (flr (n `add` 1)) - 1
  | otherwise = (flr (n `sub` 1)) + 1
```
This algorithm is guaranteed to terminate, but runs in O(n) time. We'll figure out a better algorithm that runs in O(log n) time in a later post.

Ceiling can be defined by using the identity `-(ceil x) = flr (-x)`:
```Haskell
ceil :: Dedekind -> Int
ceil n = -(flr (neg n))
```

## Testing it Out
Now let's test our code with a few test cases...
```
> ghci Dedekind.hs
...
> n = rationalDedekind 2.5
> flr n
2
> ceil n
3
> flr (n `mul` 21) -- n*21 = 2.5*21 = 52.5
52
> flr ((n `add` 0.5) `mul` 21) -- ((n+0.5)*21) = 3*21 = 63
63
> m = sqrtDedekind 2 -- sqrt(2) = 1.414213...
> flr m
1
> -- Now let's try to get the first few digits of sqrt(2)...
> flr (m `mul` 1000) -- sqrt(2) * 1000 = 1414.213...
1414
> -- It works!
> -- flr runs in O(n) time, so if we tried to run flr (m `mul` 10000) it would take a really long time
> -- Just for fun, let's try to represent (cube root of 2) + 3.5 as a Dedekind cut...
> p = Dedekind (\m -> 2 < m*m*m) (\m -> 2 > m*m*m) -- p = cube root of 2 = 1.25994...
> q = p `add` 3.5 -- cube root(2) + 3.5 = 4.75994...
> flr q
4
> flr (q `mul` 1000)
4759
> -- Success! We've gotten the first 4 digits of (cube root of 2) + 3.5
```
