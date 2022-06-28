# Extended Backus-Naur Form

## General information

- EBNF is a notation for formally describing a syntax
- EBNF rules are similar to the basic control structures in programming
  languages: sequence, decision, repetition, and recursion
- It was invented in the middle 50s, when Backus needed a notation to describe
  formal grammars of the then-popular programming languages
- It became widely adopted thanks to Peter Naur, who used it to write a complete
  ALGOL syntax
- The power of EBNF is equivalent to the power of Chomsky's type 2 notation, 
  on 3-0 scale. See: formal languages and automata theory

## Rules and Descriptions

- EBNF description is an unordered list of EBNF rules
- Each rule consists of three parts
  - A _left-hand side_, a single word, possibly with underscores
    that names the EBNF rule
  - The "is defined as" symbol, usually denoted as `<=` in formatted text or 
    `::=` in text files
  - A right-hand side, consisting of names, (special) characters and 
    combinations of the control forms described in the next section

## Control forms

- **Sequence** - Ordered list of items appearing from left to right
- **Decision** - Alternative items are separated by a `|` (pipe); only one item
  can be chosen. Same as in regexes
- **Option** - An optional item is enclosed between `[` and `]`; equivalent to
  `?` in regexes
- **Repetition** - The repeatable item is enclosed between `{` and `}`. The item
  can be repeated zero or more times; equivalent to `*` in regexes

- You can escape any of the special characters by putting it inside a box
  in the formatted text or, in text files, simply wrapping it in `''` or
  `""`
- A space can be represented with a "bucket" character or with `' '`

## Proving symbols match EBNF

- A symbol is legal if and only if all of its characters match the items
  specified in the EBNF rule description for that symbol

### Vocal proofs

- Read the sequence of characters constituting the symbol and try to match it 
  against grammar rules

### Tabular proofs

- Construct the table, where
  - The first line is the name of the EBNF rule that specifies the syntactic
    structure that the symbol will be matched against
  - The last line is the symbol that is being matched
- Each next line is derived from the previous one according to these rules
  1. Replace the name by its definition
  2. Choose an alternative
  3. Determine whether to include or omit an option
  4. Determine the number of repetitions
- Basic structure
  - Assume that `Next()` expands next rule in the current rule definition

```
+----------------------+----------------------------+
| Status               | Reason                     |
|----------------------|----------------------------|
| Rule                 | Given                      |
| Definition           | Replace "Rule" by its RHS  |
| Definition -> Next() | Replace/Choose/Include/Use | <- A new branch in the tree
| ...                  | ...                        |
| Original expression  | ...                        |
+----------------------+----------------------------+
```

### Derivation tree

- A graphical representation of a parse table
  - The EBNF rule names resides at the top, and symbol characters at the bottom
  - The downward branches illustrate the rules that allow to go from one line to
    the next one in a tabular proof

## Equivalence

- Two EBNF descriptions are equivalent if and only if they recognize exactly
  the same legal and illegal symbols

## Semantics

- The meaning of what is written
- EBNF specifies only syntax - the form of some information
- One symbol may have different contextual meanings
- Different symbols may have the same meaning

# Credits

- Richard Pattis - "[EBNF](https://www.ics.uci.edu/~pattis/ICS-33/lectures/ebnf.pdf)"

# Author

[Harutekku](https://github.com/harutekku)
