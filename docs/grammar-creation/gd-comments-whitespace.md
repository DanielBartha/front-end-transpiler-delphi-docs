# Matching Whitespace

When the parser matches comment and whitespace tokens, ideally we would like to toss them out. That way, the parser doesn't have to worry about matching optional comments and whitespace everywhere. Since DFM files cannot be conventionally commented, we will be focusing on just whitespaces.

Note that in DFM files, newlines don't act as command terminators. The structure is delimited by keywords such as *object* and the property assignments are self-terminating. For instance, `Caption = 'hello'` is unambiguous on its own because the grammar knows a property is `NAME '=' VALUE`. The newline between properties is just whitespace.

Defining discarded tokens has the same process as for non-discarded ones. We just have to indicate that the lexer should throw them out using the `skip` command, preceded by the `->` operator. Let's build on our previous example to learn how:

`../../DelphiDFM.g4`
```yaml
grammar DelphiDFM ;

// Parser Rules
file : object
object : objectElements ;

...

STRING : '\'' .*? '\'' ;
WS : [ \t\r\n]+ -> skip ;   // match 1-or-more whitespace but discard
```

That simple. For DFM files we simply skip all whitespace including newlines.