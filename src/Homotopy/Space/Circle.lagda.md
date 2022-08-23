```agda
open import 1Lab.Prelude

open import Algebra.Group

open import Data.Int.Universal
open import Data.Bool
open import Data.Int

module Homotopy.Space.Circle where
```

# Spaces: The circle

The first example of nontrivial space one typically encounters when
studying synthetic homotopy theory is the circle: it is, in a sense, the
perfect space to start with, having exactly one nontrivial path space,
which is a free group, and the simplest nontrivial free group at that:
the integers.

```agda
data S¹ : Type where
  base : S¹
  loop : base ≡ base
```

Diagramatically, we can picture the circle as being the
$\infty$-groupoid generated by the following diagram:

~~~{.quiver}
\begin{tikzpicture}
\node[draw,circle,label=below:{$\id{base}$},fill,outer sep=0.1cm, inner sep=0pt, minimum size=0.1cm] (a0) at (0, -1) {};
\draw[->] (0, 0) circle (1cm);
\node[] (loop) at (0, 0) {$\id{loop}\ i$};
\end{tikzpicture}
~~~

In type theory with K, the circle is exactly the same type as
`⊤`{.Agda}. However, with `univalence`{.Agda ident=ua}, it can be shown
that the circle has two different paths:

<!--
```
_ = ⊤
```
-->

```agda
möbius : S¹ → Type
möbius base = Bool
möbius (loop i) = ua (not , not-is-equiv) i
```

When pattern matching on the circle, we are asked to provide a basepoint
`b` and a path `l : b ≡ b`, as can be seen in the definition above. To
make it clearer, we can also define a recursion principle:

```agda
S¹-rec : ∀ {ℓ} {A : Type ℓ} (b : A) (l : b ≡ b) → S¹ → A
S¹-rec b l base     = b
S¹-rec b l (loop i) = l i
```

We call the map `möbius`{.Agda} a _double cover_ of the circle, since
the fibre at each point is a discrete space with two elements. It has an
action by the fundamental group of the circle, which has the effect of
negating the "parity" of the path. In fact, we can use `möbius`{.Agda}
to show that `loop`{.Agda} is not `refl`{.Agda}:

```agda
parity : base ≡ base → Bool
parity l = subst möbius l true

_ : parity refl ≡ true
_ = refl

_ : parity loop ≡ false
_ = refl

refl≠loop : refl ≡ loop → ⊥
refl≠loop path = true≠false (ap parity path)
```

## Fundamental group

We now calculate the loop space of the circle, relative to an
_arbitrary_ implementation of the integers: any type that satisfies
their type-theoretic universal property. We call this a _HoTT take_: the
integers are the homotopy-initial space with a point and an
automorphism. What we'd like to do is prove that the loop space of the
circle is _also_ an implementation of the integers, but it's non-trivial
to show that it is a set _directly_.

```agda
module S¹Path {ℤ} (univ : Integers ℤ) where
  open Integers univ
```

We start by defining a type family that we'll later find out is the
_universal cover_ of the circle. For now, it serves a humbler purpose: A
value in $\id{Cover}(x)$ gives a concrete representation to a path
$\id{base} \equiv x$ in the circle. We want to show that $(\id{base}
\equiv \id{base}) \simeq \bb{Z}$, so we set the fibre over the basepoint
to be the integers:

```agda
  Cover : S¹ → Type
  Cover base     = ℤ
  Cover (loop i) = ua rotate i
```

We can define a function from paths $\id{base} \equiv \id{base}$ to
integers --- or, more precisely, a function from $\id{base} \equiv x$ to
$\id{Cover}(x)$. Note that the path $\id{base} \equiv x$ induces a path
$\bb{Z} \equiv \id{Cover}(x)$, and since $\bb{Z}$ is equipped with a
choice of point, then is $\id{Cover}(x)$.

```agda
  encode : ∀ x → base ≡ x → Cover x
  encode x p = subst Cover p point
```

Let us now define the inverse function: one from integer to paths. By
the mapping-out property of the integers, we must give an equivalence
from $(\id{base} \equiv \id{base})$ to itself. Since $(\id{base} \equiv
\id{base})$ is a group, any element $e$ induces such an equivalence, by
postcomposition $x \mapsto x \cdot e$. We take $e$ to be the generating
non-trivial path, $e = \id{loop}$.

```agda
  post-loop : (base ≡ base) ≃ (base ≡ base)
  post-loop = Iso→Equiv $
    (_∙ loop) , iso (_∙ sym loop)
      (λ p → sym (∙-assoc p _ _) ·· ap (p ∙_) (∙-inv-l loop) ·· ∙-id-r p)
      (λ p → sym (∙-assoc p _ _) ·· ap (p ∙_) (∙-inv-r loop) ·· ∙-id-r p)

  loopⁿ : ℤ → base ≡ base
  loopⁿ n = map-out refl post-loop n
```

To prove that the map $n \mapsto \id{loop}^n$ is an equivalence, we
shall need to extend it to a map defined fibrewise on the cover. We
shall do so cubically, i.e., by directly pattern-matching on the
argument $x$.

<!--
```agda
  square-loopⁿ
    : (n : ℤ)
    → Square refl (loopⁿ (equiv→inverse (rotate .snd) n)) (loopⁿ n) loop
  square-loopⁿ n = composite-path→square $ sym $
    ⌜ loopⁿ (equiv→inverse (rotate .snd) n) ⌝ ∙ loop ≡⟨ ap! (map-out-rotate-inv _ _ _) ⟩
    (loopⁿ n ∙ sym loop) ∙ loop                      ≡˘⟨ ∙-assoc _ _ _ ⟩
    loopⁿ n ∙ ⌜ sym loop ∙ loop ⌝                    ≡⟨ ap! (∙-inv-l loop) ⟩
    loopⁿ n ∙ refl                                   ≡⟨ ∙-id-r _ ∙ sym (∙-id-l _) ⟩
    refl ∙ loopⁿ n                                   ∎
```
-->

When we're at the base point, we already know what we want to do: we
_have_ a function $\id{loop}^{-} : \bb{Z} \to (\id{base} \equiv
\id{base})$ already, so we can use that. For the other case, we must
provide a path $\id{loop}^{-} \equiv \id{loop}^{-}$ laying over the
loop, which we can picture as the boundary of a square.  Namely,

~~~{.quiver}
\[\begin{tikzcd}
  {\id{base}} && {\id{base}} \\
  \\
  {\id{base}} && {\id{base}}
  \arrow["{\id{loop}^{1+n}}", from=1-3, to=3-3]
  \arrow["{\id{loop}}"', from=3-1, to=3-3]
  \arrow["{\id{refl}}", from=1-1, to=1-3]
  \arrow["{\id{loop}^n}"', from=1-1, to=3-1]
\end{tikzcd}\]
~~~

```agda
  decode : ∀ x → Cover x → base ≡ x
  decode base = loopⁿ
```

We can provide such a square as sketching out an open _cube_ whose
missing face has the boundary above. Here's such a cube: the missing
face has a dotted boundary.

~~~{.quiver .tall-2}
\[\begin{tikzcd}
  \bullet &&&& \bullet \\
  & {\mathrm{base}} && {\mathrm{base}} \\
  \\
  & {\mathrm{base}} && {\mathrm{base}} \\
  \bullet &&&& \bullet
  \arrow["{\mathrm{loop}^{1+n}}"', dashed, from=2-4, to=4-4]
  \arrow["{\mathrm{loop}}", dashed, from=4-2, to=4-4]
  \arrow["{\mathrm{loop}^n}", dashed, from=2-2, to=4-2]
  \arrow[""{name=0, anchor=center, inner sep=0}, "{\mathrm{refl}}"', dashed, from=2-2, to=2-4]
  \arrow[""{name=1, anchor=center, inner sep=0}, "{\mathrm{refl}}"{description}, from=1-1, to=1-5]
  \arrow["{\mathrm{loop}^n}"{description}, from=1-5, to=5-5]
  \arrow["{\mathrm{loop}^{\mathrm{pred}(n)}}"{description}, from=1-1, to=5-1]
  \arrow["{\mathrm{loop}}"{description}, from=5-1, to=4-2]
  \arrow["{\mathrm{refl}}"{description}, from=1-1, to=2-2]
  \arrow["{\mathrm{refl}}"{description}, from=1-5, to=2-4]
  \arrow["{\mathrm{loop}}"{description}, from=5-5, to=4-4]
  \arrow["{ \mathrm{loop}}"{description}, from=5-1, to=5-5]
  \arrow["{\mathrm{base}}"{description}, draw=none, from=1, to=0]
\end{tikzcd}\]
~~~

```agda
  decode (loop i) code j = hcomp (∂ i ∨ ∂ j) λ where
    k (k = i0) → square-loopⁿ (unglue (∂ i) code) i j
    k (i = i0) → loopⁿ (equiv→unit (rotate .snd) code k) j
    k (i = i1) → loopⁿ code j
    k (j = i0) → base
    k (j = i1) → loop i
```

We understand this as a particularly annoying commutative diagram. For
example, the front face expresses the equation
$\mathrm{loop}^{n-1}\mathrm{loop} = \mathrm{loop}^{n}$. The proof is now
straightforward to wrap up:

```agda
  encode-decode : ∀ x (p : base ≡ x) → decode x (encode x p) ≡ p
  encode-decode _ = J (λ x p → decode x (encode x p) ≡ p) $
    ap loopⁿ (transport-refl point) ∙ map-out-point _ _

  encode-loopⁿ : (n : ℤ) → encode base (loopⁿ n) ≡ n
  encode-loopⁿ n = p ∙ sym (map-out-unique id refl (λ _ → refl) n) where
    p : encode base (loopⁿ n) ≡ map-out point rotate n
    p = map-out-unique (encode base ∘ loopⁿ)
      (ap (encode base) (map-out-point _ _) ∙ transport-refl point)
      (λ x → ap (encode base) {y = loopⁿ x ∙ loop} (map-out-rotate _ _ _)
          ·· subst-∙ Cover (loopⁿ x) loop point
          ·· uaβ rotate (subst Cover (loopⁿ x) point))
      n

  ΩS¹≃integers : (base ≡ base) ≃ ℤ
  ΩS¹≃integers = Iso→Equiv $
    encode base , iso loopⁿ encode-loopⁿ (encode-decode base)

open S¹Path Int-integers public
```