## Design Initiation

Considering top-down parsing, we start from the coarsest elements and descend in levels of granularity. Designing starts with natural language. Ergo, the formal definition of Delphi VCL ```.dfm``` files.

**Formal Definition** - Delphi Form Module files are proprietary file formats which store the properties, attributes and component layouts of forms, frames, and data modules.

The keyword here is **FILE(S)**.
Ergo, *file* becomes the **RULE NAME**.

```yaml
file : «sequence of component elements that are terminated by newlines» ;
```

Then, we step down a level in granularity by describing elements identified on the right side of the start rule. Typically, references to either tokens or yet-to-be-defined rules.

Tokens, are atomic elements of a parser grammar, such as words, punctuation, operators, etc.
Rule references, are other language structures which need to be broken down into more detail, such as *object*.

Descending further into the next granularity level, for instance, *object* should represent a sequence of elements which need to be contained within *file*. For example: `Caption`, `Left`, `Top`, `Width`, etc.

Then, a pseudo-code of Parser Rules looks like this:

```yaml
file : «sequence of terminals such as object or inherited, that are terminated by newlines» ;
object : «sequence of objectElements» ;
objectElements : «sequence of alternative terminal tokens» ;
```

Descending further, now we know what the terminal tokens of *objectElements* should contain. Therefore, we define Lexer and Parser Rules for non-terminal and terminal tokens respectively. For instance:

`../../DelphiDFM.g4`
```yaml
grammar DelphiDFM ;

// Parser Rules
file : object
object : objectElements ;
objectElements : CAPTION | LEFT '=' INT | ... ;

// Lexer Rules
CAPTION : 'Caption' ;
LEFT : 'Left' ;
INT : [0-9]+ ;
ID : [a-zA-Z]+ ;
STRING : '\'' .*? '\'' ; 	// STRING reads as anything between single quotes
```

Continuing until no further rules can be defined.

**Notes**

- grammars always start with a grammar header. In our case, the grammar is called `DelphiDFM`, and must match the file name: `DelphiDFM.g4`.

- the `+` in `INT` and `ID` above, represent an ANTLR specific subrule which describes an arbitrarily long sequence. Essentially means to encode a sequence of one or more elements.

- the zero-or-more operator `*` specifies that a list can be empty. Example: `INT*`. This operator is analogous to a loop in programming languages, ANTLR-generated parsers implement them in this way as well.

- the alternative operator `|` specifies alternative options of non-terminals and terminals.

- the `?` subrule is a special zero-or-one sequence, and specifies optional constructs. For instance, `given a token 'x', if 'x?' then match 'x' or skip it`.

- single quotes in ANTLR are reserved, therefore they need to be escaped like so: `'\''`.

- ANTLR resolves potential ambiguity between lexical rules by favoring the rule *specified first*. Take the following example:

`../../AmbiguousDelphiDFM.g4`
```yaml
grammar AmbiguousDelphiDFM ;

file : functionDeclaration | FOR | ... ;
functionDeclaration : 'for' ;

...

// Lexer Rules
ID  : [a-zA-Z]+ ;
FOR : 'for' ;

...
```

In the example above, since `FOR` can be evaluated using `ID` as well, we have an ambiguity conflict. The correct solution is to specify rules in order of precedence:

```yaml
grammar NoLongerAmbiguousDelphiDFM ;

file : functionDeclaration | FOR | ... ;
functionDeclaration : 'for' ;

...

// Lexer Rules
FOR : 'for' ;
ID  : [a-zA-Z]+ ;

...
```

This way, the literal *for* will always be tokenized as `FOR`, and never as `ID` in the grammar.