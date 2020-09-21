# Writing a Calculus of Constructions Type Checker
intro intro intro
mention de bruijn
## Types
First, let's introduce a type for variables.
This isn't strictly necessary, but it makes our code more readable later.
Since we're using De Bruijn indexing, variables are just integers:
```Haskell
type Var = Int
```
Now we can write a type that defines CoC terms.
```Haskell
data Term =
```
terms can be a `*`...
```Haskell
Star |`
```
...or a lambda
(This one will have 2 arguments.
One is the type of the lambda's argument.
The other is what it returns.
So for example, `λa:b.c` would be represented in our type as `Lambda b c`.
Note that there's no Var argument, since De Bruijn indexes always start at 0 and aren't named.)...
```Haskell
    Lambda Term Term |
```
...or a forall
(This has 2 arguments for the same reason as Lambda)...
```Haskell
    Forall Term Term |
```
...or calling one term with another (`Called a b` represents `(a b)`)...
```Haskell
    Called Term Term |
```
...or a variable...
```Haskell
    VarTerm Var
```
## `typeof`, First Attempt
Now let's write a function that gets the type of a term.
If the term has no type, our function will return Nothing.
```Haskell
typeof :: Term -> Maybe Term
```
The case of `*` is easy: `*` has no type.
```Haskell
typeof Star = Nothing
```
Foralls are also easy: their type is `*`:
```Haskell
typeof (Forall _ _) = Just Star
```
Lambdas get a bit trickier: we need to get the type of the returned value first
```Haskell
typeof (Lambda a b) = do
    typeOfB <- typeof b
    return (Forall a typeOfB)
```
For `Called`, we use the rule that (a b):d if a:(c -> d) and b:c.
```Haskell
typeof (Called a b) = do
    typeOfA <- typeof a
    typeOfB <- typeof b
    case typeOfA of
        (Forall c d) -> if (typeOfB == c) then Just d else Nothing
        _        -> Nothing
```
Note that we haven't defined equality of `Term`s yet. We'll deal with this later in the post.

Finally, we get to `typeof (VarTerm a)` and we're left with a dilemma.
We don't know the context.
What we need to do is keep track of the types of variables along the way.
This is what we're going to try to do next.

## The Environment
We need to keep track of the types of each of the variables.
Ideally, this would be a mapping from `Var -> Term`.
Since `Var`s are just non-negative `Int`s, we can use a list to do this:
```Haskell
type Env = [Term]
```
We'll need 2 functions for interacting with the environment: one that looks for a variable in it, and another that adds a variable to it.

Looking for a variable in the environment is pretty easy.
Just to be on the safe side, we'll make this function return `Nothing` if we look for a variable that's out of range.
```Haskell
lookup :: Env -> Var -> Maybe Term
lookup env var = if (var < length env) then Just (env !! var) else Nothing
```
Adding a variable to an environment is slightly harder.
We're using De Bruijn, so when we add something to the environment,
we also want to increment all the variables in the environment.
```Haskell
addToEnv :: Env -> Term -> Env
addToEnv env a = map incrementVars (a:env)
```
Making this incrementVars function is pretty self-explanatory, though:
```Haskell
incrementVars :: Term -> Term
incrementVars Star         = Star
incrementVars (Lambda a b) = Lambda (incrementVars a) (incrementVars b)
incrementVars (Forall a b) = Forall (incrementVars a) (incrementVars b)
incrementVars (Called a b) = Called (incrementVars a) (incrementVars b)
incrementVars (VarTerm a)  = VarTerm (a+1)
```
## `typeof`, Second Attempt
Now that we've defined tools for interacting with the environment, we can take another try at making our `typeof` function:
```Haskell
typeof :: Env -> Term -> Maybe Term
typeof env Star = Nothing
typeof env (Forall _ _) = Just Star
typeof env (Lambda a b) = do
    typeOfB <- typeof (addToEnv env a) b
    return (Forall a typeOfB)
typeof env (Called a b) = do
    typeOfA <- typeof env a
    typeOfB <- typeof env b
    case typeOfA of
        (Forall c d) -> if (typeOfB == c) then Just d else Nothing
        _        -> Nothing
typeof env (VarTerm a) = lookup env a
```
Our definition here has 2 problems:
we don't check if everything is valid before returning its type (most notably Forall), and
we still don't know what equality of terms means.
So let's deal with these problems next.
## Checking Validity
Let's define what a valid term means.
We plan on having typeof check that a term is valid and *then* returning its type,
so this is an easy way to check for a term's validity.
The one exception is Star: Star is valid, but has no type.
Encoding this, we get:
```Haskell
valid :: Env -> Term -> Bool
valid _ Star = True
valid env t = case (typeof env t) of
    Just _ -> True
    Nothing -> False
```
## Calling
Before we get to checking equality of terms, we need to figure out how to simplify terms.
`Called`s are the only type that we need to simplify, since everything else is pretty much simplified already.

Let's start writing a function `call` and figure out what helper functions we'll need along the way.
```Haskell
call :: Env -> Term -> Term -> Maybe Term
```
`call` will take 2 terms and return what happens when the first is called by the second (if it's valid).
First, let's take care an easy case.
If we have a `Called` in the first argument, we simplify that and try calling again:
```Haskell
call env (Called a b) c = do
    ab <- call env a b
    Just $ call env ab c
```
Now let's look at the real beef of our function:
calling a lambda.
```Haskell
call env (Lambda a b) c = do
    tc <- typeof env c
    if (equals env tc a)
        then Just $ replace 0 c b
        else Nothing
```
We see 2 new functions here: `equals` and `replace`.
`equals env a b` checks if `a` and `b` are equal in environment `env`; we'll deal with this later.
`replace v a b` replaces variable `v` with `a` in `b`.
Let's get the last case of `call` out of the way, and then deal with this function.
```Haskell
call _ _ _ = Nothing
```
## Replacing Variables
Replacing variables isn't too difficult with De Bruijn indexing; let's look at the easy cases first:
```Haskell
replace :: Var -> Term -> Term -> Term
replace var x Star = Star
replace var x (Called a b) = Called (replace var x a) (replace var x b)
```
For `replace var x (VarTerm v)`, we check if this is the var we're looking to replace.
If it is, we replace it.
Otherwise, we leave it alone:
```Haskell
replace var x (VarTerm v)
    | var == v = x
    | otherwise = VarTerm v
```
`Forall` and `Lambda` are a bit harder.
Since we're using De Bruijn, we need to increment variables before we go into subterms;
this'll need another helper function:
```
replace var x (Forall t a) = Forall (replace var x t) (replace (var+1) (incrementVars x) a)
replace var x (Pi t a) = Pi (replace var x t) (replace (var+1) (incrementVars x) a)
```
The new helper function, incrementVars, should be pretty self-explanatory.
It increments all variables in a term and returns the result.
```Haskell
incrementVars :: Term -> Term
incrementVars (VarTerm a) = VarTerm (a+1)
incrementVars Star = Star
incrementVars (Forall a b) = Forall (incrementVars a) (incrementVars b)
incrementVars (Lambda a b) = Lambda (incrementVars a) (incrementVars b)
incrementVars (Called a b) = Called (incrementVars a) (incrementVars b)
```
We've written all the helper functions we needed for `call`, so let's move on to `equals`.
## Equality of Terms
