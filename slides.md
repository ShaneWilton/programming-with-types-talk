class: middle center

# Programming with Types
## Quinn Wilton

---

# What is a type system?

 - Rules used by a programming language to determine illegal program states

 - Maps types to expressions

 - Examples, in Haskell:
   - `String`, a string
   - `(Integer, Integer)`, a pair of elements, each an integer
   - `String -> Integer`, a function which takes a string as input, and returns an integer

---
class: middle

# ...Dynamic

Type systems can be dynamic, and checked at runtime, like in Ruby:

```ruby
> 1 + "test"
TypeError (String can't be coerced into Integer)
```

---
class: middle

# ...Static

Or they can be static, and checked at compile time, as in Haskell.
```haskell
add :: Integer -> Integer -> Integer
add x y = x + y

main = putStrLn . show $ add 5 "test"
```
```haskell
❯ ghc static-example.hs
[1 of 1] Compiling Main             ( static-example.hs, static-example.o )

static-example.hs:4:32: error:
    • Couldn't match expected type ‘Integer’ with actual type ‘[Char]’
    • In the second argument of ‘add’, namely ‘"test"’
      In the second argument of ‘($)’, namely ‘add 5 "test"’
      In the expression: putStrLn . show $ add 5 "test"
  |
4 | main = putStrLn . show $ add 5 "test"
  |                                ^^^^^^
```

---
class: center middle

# ALL languages have type systems

---

# Javascript
Dynamic, and very permissive:
```javascript
> {} + []
0

> [] + {}
"[object object]"

> [] + []
""

> Array(16).join("wat" - 1)
NaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaN
```

---

# Elixir

Dynamic, but less permissive:
```elixir
iex(1)> {} + []
** (ArithmeticError) bad argument in arithmetic expression: {} + []
    :erlang.+({}, [])

iex(2)> IO.puts {1, 2}
** (Protocol.UndefinedError) protocol String.Chars not implemented for {1,2}
    (elixir) lib/string/chars.ex:3: String.Chars.impl_for!/1
    (elixir) lib/string/chars.ex:22: String.Chars.to_string/1
    (elixir) lib/io.ex:553: IO.puts/2

iex(3)> IO.puts inspect({1, 2})
{1, 2}
:ok
```

---
class: center middle

# Regardless of the language, you can always think in types

---
class: center middle

# Why?

---
class: middle

# Domain Modeling with Types
Domain Model: a conceptual model of the domain that incorporates both behavior and data.

- The real world is messy

- Easier to reason about a simpler domain model, that focuses on:
  - Business Requirements
  - Security Concerns
  - Data Invariants

- Why not encode these into the type system?

---
class: center middle

# Algebraic Data Types

---
# Product Types

Tuples and Records.

Elixir has record types, but they mostly exist for interopability with Erlang. Use maps / structs instead.

Elixir:
```elixir
@type coordinate :: {integer, integer}

@type person :: %{
  name: String.t(),
  age: integer()
}
```

Haskell:
```haskell
data Coordinate = Coordinate Integer Integer deriving (Show)

data Person = Person { name :: String
                     , age :: Integer} deriving (Show)
```

---
class: middle

# Sum Types
A tagged union, representing the choice of one type out of many.


Elixir:
```elixir
@type color :: :red | :green | :blue

@type division_result :: :division_by_zero | {:success, float()}
```

Haskell:
```haskell
data Color = Red | Green | Blue deriving (Show)

data DivisionResult = DivisionByZero
                    | Success Double deriving (Show)
```

---
class: middle

# Back to Algebraic Data Types
Composite types made up of the composition of product types and sum types.

Some examples:
```haskell
data List a = Empty
            | Cons a (List a) deriving (Show)

data Expression = Number Int
                | Add Expression Expression
                | Minus Expression Expression
                | Mult Expression Expression
                | Divide Expression Expression deriving (Show)

data NaturalNumber = Zero
                   | Successor NaturalNumber deriving (Show)
```

---

# Trees!

```
                                0
                              /   \
                             1     2
```

Elixir:
```elixir
@type tree :: :leaf | {:node, integer, tree(), tree()}

tree =
  {:node,0,
    {:node, 1,
      :leaf,
      :leaf},
    {:node, 2,
      :leaf,
      :leaf}}
```

Haskell:
```haskell
data Tree = Leaf | Node Integer Tree Tree deriving (Show)

tree = Node 0 (Node 1 Leaf Leaf) (Node 2 Leaf Leaf)
```

---

# Tree Traversal

```elixir
defmodule Tree do
  def map(tree, f) do
    case tree do
      :leaf ->
        :leaf
      {:node, value, left, right} ->
        {:node, f.(value), map(left, f), map(right, f)}
    end
  end
end

tree =
  {:node,0,
    {:node, 1,
      :leaf,
      :leaf},
    {:node, 2,
      :leaf,
      :leaf}}

Tree.map(tree, fn val -> val + 1 end)
```

---
class: middle

# ...and in Haskell!

```haskell
data Tree = Leaf | Node Integer Tree Tree deriving (Show)

tree_map :: Tree -> (Integer -> Integer) -> Tree
tree_map Leaf f = Leaf
tree_map (Node value left right) f =
  Node (f value) (tree_map left f) (tree_map right f)
```

Then in a REPL:
```haskell
> t = Node 0 (Node 1 Leaf Leaf) (Node 2 Leaf Leaf)
> tree_map t (+ 1)
Node 1 (Node 2 Leaf Leaf) (Node 3 Leaf Leaf)
```

---
class: middle

# Optional Types
- No concept of `nil` or `null` in most languages like Haskell!

- Uninhabited values need to be encoded into the type system.

- Introducing... `Maybe`

```haskell
data Maybe a = Nothing | Just a deriving (Show)

first :: [a] -> Maybe a
first [] = Nothing
first (x:xs) = Just x
```

---
class: middle

# Impossible States
What's wrong with this design?

```haskell
data User = User { username :: String
                 , name :: String
                 , phoneNumber :: String
                 , emailAddress :: String
                 , emailVerified :: Boolean
                 , paymentType :: String
                 , paypalId :: String
                 , stripeId :: String }
```

---
class: middle

# Name is optional!
```haskell
data User = User { username :: String
                 , name :: Maybe String
                 , phoneNumber :: String
                 , emailAddress :: String
                 , emailVerified :: Boolean
                 , paymentType :: String
                 , paypalId :: String
                 , stripeId :: String }
```

---
class: middle

# Mutally exclusive fields!
```haskell
data ContactInfo = PhoneNumber String
                 | EmailAddress { address :: String
                                , verified :: Boolean }

data User = User { username :: String
                 , name :: Maybe String
                 , contactInfo :: ContactInfo
                 , paymentType :: String
                 , paypalId :: String
                 , stripeId :: String }
```

---
class: middle

# Email Related Business Logic!
```haskell
data UnverifiedEmail = UnverifiedEmail String
data VerifiedEmail = VerifiedEmail String

data EmailContactInfo = Unverified UnverifiedEmail
                      | Verified VerifiedEmail

data ContactInfo = PhoneNumber String
                 | EmailContactInfo

...

sendEmail :: VerifiedEmail -> <some result>
sendEmail email = ...

newEmail :: String -> UnverifiedEmail
newEmail email -> UnverifiedEmail email

verifyEmail :: UnverifiedEmail -> VerifiedEmail
verifyEmail email = VerifiedEmail email
```

---
class: middle

# Payment Methods!
```haskell
data PaymentMethod = Invoice
                   | PayPal { id :: String }
                   | Stripe { id :: String }

...

data User = User { username :: String
                , name :: Maybe String
                , contactInfo :: ContactInfo
                , paymentMethod :: PaymentMethod

...

chargeUser User { paymentMethod = Invoice } =
  doNothing

chargeUser User { paymentMethod = (Paypal paypal) } =
  handlePaypal paypal

chargeUser User { paymentMethod = (Stripe stripe) } =
  handleStripe stripe
```

---
class: middle

# Now the types tell us...
- Which fields are optional

- Which fields are mutually exclusive

- How types can be transformed between each other

- What capabilities different types have

---
class: center middle

# You can still domain model in Elixir!

---
class: middle
# Modeling Failures
What's wrong with this design?

```haskell
data ChatHistory = ChatHistory { messages :: [String]
                               , loading :: Boolean }

initialChatHistory = ChatHistory { messages = []
                                 , loading = false }
```

---
class: middle

# Woah.
```haskell
data RemoteData e a = NotRequested
                    | Loading
                    | Failure e
                    | Success a

data ChatHistory = ChatHistory (RemoteData Http.Error [String])

renderRemoteData :: RemoteData e a -> String
renderRemoteData NotRequested = "Not requested yet!"
renderRemoteData Loading = "Loading..."
renderRemoteData (Failure e) = "Error: " ++ (show e)
renderRemoteData (Success a) = "Fetched: " ++ (show a)
```

---
class: center middle
# Types as Security!

---
class: center middle
# Cross-Site Scripting?

---
class: middle
# Cross-Site Scripting!

```haskell
data UserInput = Unsafe String
               | Escaped String

escape :: UserInput -> UserInput
escape (Unsafe input) = Escaped (htmlEscape input)
escape (Escaped input) = Escaped input

interpolate :: Template -> UserInput -> Document
interpolate t (Unsafe input) = interpolate t (escape input)
interpolate t (Escaped input) = ...
```

---
class: center middle
# Information Disclosure

---
class: middle
# Information Disclosure!
```haskell
data Public = Public String
data Private = Private String

data Variable a = Variable String a

data UnprivilegedView =
  UnprivilegedView { template :: String
                   , variables :: [Variable Public]}

data PrivilegedView =
  PrivilegedView { template :: String
                 , public :: [Variable Public]
                 , private :: [Variable Private] }

data View = Privileged PrivilegedView
          | Unprivileged UnprivilegedView
```

---
class: center middle

# You get the point!

---
class: middle

# Bonus: Other Type Systems!
Haskell has a strong type system, but it doesn't have the strongest. There exist other formulations of type systems that can express constraints not possible in Haskell.

- `Idris` is a dependently typed language, based on Haskell, in which types are first class citizens, and can be used in computations directly
- `Coq` is dependently typed implementation of higher-order type theory, used as an interactive theorem prover
- `Session Types` are a new branch of type theory, used for type checking communication protocols
- `Linear Types`, implemented using linear logic, can be used to type check the management of resources, such as memory allocations, providing compile-time memory safety without the overheard of a tracing garbage collector
- `Pony`, an experimental actor-model based language, models capabilities using its type system to enable the creation of secure systems and model secure information flow

---
class: middle

# Futher Reading / Watching
- Making Impossible States Impossible
  - https://www.youtube.com/watch?v=IcgmSRJHu_8
- How Elm Slays a UI Antipattern
  - http://blog.jenkster.com/2016/06/how-elm-slays-a-ui-antipattern.html
- Domain Modeling Made Functional
  - https://www.youtube.com/watch?v=Up7LcbGZFuo
- Type First Development
  - http://tomasp.net/blog/type-first-development.aspx/
- Types and Programming Languages
  - https://www.cis.upenn.edu/~bcpierce/tapl/
