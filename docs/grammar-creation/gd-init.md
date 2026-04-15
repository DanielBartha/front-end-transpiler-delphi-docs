# Design Initiation

Considering top-down parsing, we start from the coarsest elements and descend in levels of granularity. Designing starts with natural language. Ergo, the formal definition of Delphi VCL ```.dfm``` files.

**Formal Definition** - Delphi Form Module files are proprietary file formats which store the properties, attributes and component layouts of forms, frames, and data modules.

The keyword here is **FILE(S)**. Thus, we specify *file* as our entry point, becoming the **RULE NAME**.

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

Continuing until no further rules can be defined. Or until the first valid iteration is developed.

**Notes**

- Grammars always start with a grammar header. In our case, the grammar is called `DelphiDFM`, and must match the file name: `DelphiDFM.g4`.
- *ANTLRv4* allows parser and lexer rules to exist in the same `.g4` file, as opposed to previous versions.
- The `+` in `INT` and `ID` above, represent an ANTLR specific subrule which describes an arbitrarily long sequence. Essentially means to encode a sequence of one or more elements.
- The zero-or-more operator `*` specifies that a list can be empty. Example: `INT*`. This operator is analogous to a loop in programming languages, ANTLR-generated parsers implement them in this way as well.
- The alternative operator `|` specifies alternative options of non-terminals and terminals.
- The `?` subrule is a special zero-or-one sequence, and specifies optional constructs. For instance, `given a token 'x', if 'x?' then match 'x' or skip it`.
- Single quotes in ANTLR are reserved, therefore they need to be escaped like so: `'\''`.

# The Ambiguity Conflict

*ANTLRv4* resolves ambiguity conflicts between lexical rules by favoring the rule **specified first**. 

In the example below, `FOR` can be evaluated using `ID` as well. Thus, we have an ambiguity conflict due to order of precedence. In this case, the literal *for* will be tokenized as `ID` first, instead of the desired `FOR`: 

`../../AmbiguousDelphiDFM.g4`
```yaml
grammar AmbiguousDelphiDFM ;

// Parser Rules
file : functionDeclaration | FOR | ... ;
functionDeclaration : 'for' ;

...

// Lexer Rules
ID  : [a-zA-Z]+ ;
FOR : 'for' ;

...

```

The suitable solution, is to specify rules with correct order of precedence:

```yaml
grammar Non-AmbiguousDelphiDFM ;

file : functionDeclaration | FOR | ... ;
functionDeclaration : 'for' ;

...

// Lexer Rules
FOR : 'for' ;
ID  : [a-zA-Z]+ ;

...

```

This way, the literal *for* will always be tokenized as `FOR`, and never as `ID` in the grammar.
