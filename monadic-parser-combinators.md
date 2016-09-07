# Monadic Parser Combinators

Paper: http://www.cs.nott.ac.uk/~pszgmh/monparsing.pdf by Hutton + Meijer

[Video](https://channel9.msdn.com/Series/C9-Lectures-Erik-Meijer-Functional-Programming-Fundamentals/C9-Lectures-Dr-Erik-Meijer-Functional-Programming-Fundamentals-Chapter-8-of-13)

A functional approach to building parsers is to:
- model parsers as functions
- model combinators as higher order functions that implement sequencing (bind), choice, repetition

### Parser type
The Parser data type is `type Parser a = [(a, String)]` "a" represents the parsed value type (some sort of a Tree for example), and "String" is the remaining unparsed text. An empty list [] represents failure to parse. And a list of many tuples represents all the different ways to parse the input String (for when you have abigious grammer rules)

a function that parses characters has the type: `charParser :: String -> Parser Char`

### Primitive parsers
These represent the building blocks for combinator parsing: ("a" represents any data type like Char, String, or some other customized data type)

1) result parser, it doesn't care about the input string and always returns
```haskell
result :: a -> Parser a
result v = \inp -> [(v, inp)]
```

2) zero parser for when parsing fails
```haskell
zero :: Parser a
zero = \inp -> []
```

3) item parser that parses Char, Num, etc
```haskell
item :: Parser Char
item = \inp -> case inp of
   [] -> []
   (x:xs) -> [(x, xs)]
```

### Parser combinators
- glue primitive parsers to form more useful ones

1) bind combinator
sequencing, applying one parser after another
```haskell
bind :: Parser a -> (a -> Parser b) -> Parser b
bind p f = \inp -> concat [f v inp' | (v, inp') <- p inp]
```

We can build a more complex parser using bind, to glue more than 1 parser together:
```haskell
p1 `bind` \x1 ->
p2 `bind` \x2 ->
pn `bind` \xn ->
result (f x1 x2 xn)
```

2) plus combinator
```haskell
plus :: Parser a -> Parser a -> Parser
plus = \inp -> (p inp ++ q inp)
```

Apply the string to be parsed to two parsers, and return the result from both or either (if one parser fails it will return an empty list). This combinator fails only if the two parsers fail, the bind combinator will fail as soon as a parser fails.

Example of using the plus combinator: if we are trying to match a letter and don't care about its case
```haskell
letter :: Parser Char
letter = lower `plus` upper
```

### Repetition combinators
