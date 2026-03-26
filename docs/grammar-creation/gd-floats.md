# Matching Floating-Point Numbers

Floating-point numbers are more complex to represent and additional rules will need to be implemented because of them. Take the example below:

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

fragment
DIGIT : [0-9] ;		

STRING : '\'' .*? '\'' ;
```

**Notes**

- the order of precedence in the lexer rules, specifically `FLOAT` and `INT`. If their order would be reversed, an ambiguity conflict would occur. For example, if the lexer sees the sequence `123` it could match either. Thus, if `INT` is defined first, the lexer will always prefer `INT` and never reach `FLOAT`.

- the (now) correct specification of `CAPTION` in *objectElements*.

- the addition of the helper rule `DIGIT`, to not write `[0-9]` everywhere.

- `DIGIT` is prefixed with `fragment` to let ANTLR know that the rule will be used only by other lexical rules. It is not a token in and of itself.