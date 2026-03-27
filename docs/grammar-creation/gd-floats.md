# Matching Floating-Point Numbers

Floating-point numbers are more complex to represent, and additional rules will need to be implemented because of them. Take the example below:

`../../DelphiDFM.g4`
```yaml
grammar DelphiDFM ;

// Parser Rules
file : object
object : objectElements ;
objectElements : CAPTION '=' STRING | LEFT '=' INT | TOP '=' FLOAT | ... ;

// Lexer Rules
CAPTION : 'Caption' ;
LEFT : 'Left' ;
TOP : 'Top' ;

ID : [a-zA-Z]+ ;
FLOAT : DIGIT+ '.' DIGIT* | '.' DIGIT+ ;
INT : DIGIT+ ;

fragmented DIGIT : [0-9] ;		
STRING : '\'' .*? '\'' ;
```

**Note:**

- The order of precedence in the lexer rules, specifically `FLOAT` and `INT`. If their order would be reversed, an ambiguity conflict would occur. For example, if the lexer sees the sequence `123` it could match either. Thus, if `INT` is defined first, the lexer will always prefer `INT` and never reach `FLOAT`.
- The separation between property elements and their values, through the `=` operator.
- The addition of the helper rule `DIGIT`, to avoid writing out `[0-9]` everywhere.
- `DIGIT` is prefixed with `fragmented` to let *ANTLR* know that the rule will be used only by other lexical rules (will be elaborated further down the line).