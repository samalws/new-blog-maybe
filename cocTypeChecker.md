# Writing a Calculus of Constructions Type Checker
intro intro intro
mention de bruijn
## Types
First, let's introduce a type for variables.
This isn't strictly necessary, but it makes our code more readable later.
Since we're using De Bruijn indexing, variables are just integers:
```
type Var = Int
```
Now we can write a type that defines CoC terms.
```
data Term =
```
terms can be a `*`...
```
Star |`
```
...or a lambda
(This one will have 2 arguments.
One is the type of the lambda's argument.
The other is what it returns.
So for example, `λa:b.c` would be represented in our type as `Lambda b c`.
Note that there's no Var argument, since De Bruijn indexes always start at 0 and aren't named.)...
```
    Lambda Term Term |
```
...or a forall
(This has 2 arguments for the same reason as Lambda)...
```
    Forall Term Term |
```
...or calling one term with another (`Called a b` represents `(a b)`)...
```
    Called Term Term |
```
...or a variable...
```
    VarTerm Var
```
## `typeof`, First Attempt
Now let's write a function that gets the type of a term.
If the term has no type, our function will return Nothing.
```
typeof :: Term -> Maybe Term
```
The case of `*` is easy: `*` has no type.
```
typeof Star = Nothing
```
Foralls are also easy: their type is `*`:
```
typeof (Forall _ _) = Just Star
```
Lambdas get a bit trickier: we need to get the type of the returned value first
```
typeof (Lambda a b) = do
    typeOfB <- typeof b
    return (Forall a typeOfB)
```
For `Called`, we use the rule that (a b):d if a:(c -> d) and b:c.
```
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
```
type Env = [Term]
```
We'll need 2 functions for interacting with the environment: one that looks for a variable in it, and another that adds a variable to it.

Looking for a variable in the environment is pretty easy.
Just to be on the safe side, we'll make this function return `Nothing` if we look for a variable that's out of range.
```
lookup :: Env -> Var -> Maybe Term
lookup env var = if (var < length env) then Just (env !! var) else Nothing
```
Adding a variable to an environment is slightly harder.
We're using De Bruijn, so when we add something to the environment,
we also want to increment all the variables in the environment.
```
addToEnv :: Env -> Term -> Env
addToEnv env a = map incrementVars (a:env)
```
Making this incrementVars function is pretty self-explanatory, though:
```
incrementVars :: Term -> Term
incrementVars Star         = Star
incrementVars (Lambda a b) = Lambda (incrementVars a) (incrementVars b)
incrementVars (Forall a b) = Forall (incrementVars a) (incrementVars b)
incrementVars (Called a b) = Called (incrementVars a) (incrementVars b)
incrementVars (VarTerm a)  = VarTerm (a+1)
```
## `typeof`, Second Attempt
Now that we've defined tools for interacting with the environment, we can take another try at making our `typeof` function:
```
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
we don't check if everything is valid before returning its type (most notably Forall),
and we still don't know what equality of terms means.
So let's deal with these problems next.
## Equality of Terms
