# Calculus of Constructions, for Math Nerds
*This post is aimed at introduced the Calculus of Constructions (CoC) to math nerds.
A decent knowledge of how proofs work is assumed, but the rest will be introduced here.*
As a math nerd (presumably, since you're reading this), you've seen many proofs.
But how do you know these proofs are correct?
bla bla bla etc introduction
## Rule 1: if a is a valid term, and a has type b, then b is true and a is a proof of b
filler text
## Rule 2: ∀a:b.c means "for all a of type b, c"
filler text
## Rule 3: λa:b.c is a function taking argument a of type b, and returning c
filler text
## Rule 4: if d:b, then ((λa:b.c) d) = (c, with all occurrences of "a" replaced with d)
filler text
## Rule 5: if b = e and (c given a:b) = (f given d:e), then (λa:b.c):(∀d:e.f)
filler text
## Rule 6: if a:(∀c:d.e) and b:c, then (a b):e
TODO what about replacement??
filler text
## Rule 7: (∀a:b.c):\*
filler text
## Rule 8: \* has no type
filler text
## Other proof symbols, in CoC
filler text

(a → b) := ∀x:a.b

False := ∀a:*.a

¬a := a → False

(a ∧ b) := ∀c:*.(a → (b → c)) → c

(a ∨ b) := ∀c:*.(a → c) → ((b → c) → c)

(a ↔ b) := (a → b) ∧ (b → a)

∃a:b.c := ¬(∀a:b.¬c)

(a = b) := ∀c:*.(∀f: ) → () uhhh
