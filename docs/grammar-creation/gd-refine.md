# Refining the Grammar

Through the previous sections we have described all necessary points needed for designing an initial ANTLR-specific grammar for Delphi DFM files.

Now it is time to actually put things in practice and refine the previous examples into a usable DFM grammar, since some things are missing. For instance, how do we know when an **object** block ends?; What about inheritance? What happens with nested elements?. We will be answering all these questions and more.

Let's closely examine the refined version of the grammar below. Note that alignment has changed, due to readability purposes and to adhere to *ANTLR's* conventions.

`../../DelphiDFM.g4`
```yaml
grammar DelphiDFM ;

file             : block+ EOF 
                 ;

// Parser Rules
block            : ('inherited' | 'object') ID ':' ID propertyBlock* block* 'end'
                 ;

propertyBlock    : SUBPROPERTY '=' value                                            
                 | ID '=' value                                                     
                 ;

value            : INT                                                              
                 | FLOAT                                                            
                 | STRING                                                           
                 | ID                                                               
                 | ARRAY                                                            
                 | collectionItems                                                  
                 ;

collectionItems  : '<' item* '>' ;
item             : 'item' propertyBlock* 'end' ;

// Lexer Rules
INHERITED        : 'inherited' ;
OBJECT           : 'object' ;
END              : 'end' ;
ITEM             : 'item' ;

SUBPROPERTY      : ID_FRAG '.' ID_FRAG ('.' ID_FRAG)? ;
ARRAY            : '[' .*? ']' ;
ID               : ID_FRAG ;

FLOAT            : '-'? (DIGIT+ '.' DIGIT* 
                 | '.' DIGIT+) 
                 ;

INT              : '-'? DIGIT+ ;

fragment ID_FRAG : [a-zA-Z][a-zA-Z0-9_]* ;
fragment DIGIT   : [0-9] ;

STRING           : '\'' .*? '\'' ;
WS               : [ \t\r\n]+ -> skip ;
```

Many specifications have changed from the initial examples. Let's decompose them.

## Parser Rules

### File Rule

First, we recognize that in essence, DFM files are composed of blocks containing all set properties, component positions/dimensions, event handlers, etc. Therefore, our *file* needs to contain at least one or more of these blocks, through `block+ EOF`. By including `EOF` (End of File), we explicitly specify that we want to parse the entire file. Omitting `EOF` would mean that it is acceptable to only parse a portion of the input.

**Note**: The `EOF` rule means "Parse as many *block* elements as possible, and then stop." In other words, this rule will never attempt to recover from a syntax error because it will always assume that the syntax error is part of some syntactic construct that's beyond the scope of the *file* rule. Syntax errors will not even be reported, because the parser will simply stop. 

### Block Rule

Since DFM blocks can start with one of two options (*inherited* or *object*), we specify a choice for the parser to opt for one. Either of the options, always continues with a string, followed by a colon, followed by another string.

Then, we follow it up with two optional parser rules. Meaning, that there *could be zero-or-more* blocks of properties or additional blocks following. By including `block*`, we are accounting for nested elements. 

Finally, we conlude a `block` parser rule, by specifying `end`, which is our reserved rule defined in the lexer rules. Hence, all blocks in DFM files conclude with an *end* specifier.

### The Property Block

All values in DFM files are assigned to property and sub-property declarations, separated by the `=` operator. To simplify parsing them, we assign the parser rule `value` to both `ID` and `SUBPROPERTY`. In the lexer rules, we classify `SUBPROPERTY` as having two or more strings concatenated by **dots**.

### Collection Items & Item

An important observation here, is the inclusion of list collection elements, through the `collectionItems` rule. These lists are enclosed in `<>` operators, but also between keywords of `item` and `end`. To handle this, we reserve the keyword `item` as a parser rule, and provide optional property blocks, then end it with the reserved `end` keyword. Then we simply include the `item` rule as optional within `collectionItems`, enclosed between the `<>` operators. Finally, we pass it to property block declarations as the `value` rule, which contains `collectionItems` as an alternative.

## Lexer Rules

Besides the obvious rules such as `INHERITED` and `OBJECT`, we have some interesting things going on here.

The `value` parser rule specifies alternatives between various lexer rules, such as `INT`, `FLOAT`, `ARRAY`, etc. We have a noticeable overlap here, involving `STRING` and `ID`. That is because we need to make the distinction between normal strings and string literals.

Notice that the `ID` rule's definition changed from the previous simplistic `[a-zA-Z]+` to `ID_FRAG`. The helper rule *fragmented*, preceding `ID_FRAG`, let's *ANTLR* know, that `ID_FRAG` will only be used by other lexical rules. *Fragmented* rules are not tokens themselves. By using this helper rule, we are able to simply pass `ID_FRAG` wherever we need its value, instead of writing out `[a-zA-Z][a-zA-Z0-9_]*` everywhere.

The very same *fragmented* rule is applied to `DIGIT` as well, for the same purpose.

**Side-note**: there is a caveat to the `ARRAY` rule. It is only able to handle commas as long as the array itself does not span multiple lines. Since DFM files do not contain such large arrays typically, this rule specification suffices.

Additionally, notice the `'-'?` specification for the `FLOAT` and `INT` rules. We use it to optionally account for negative values. This way however, does not account for potential whitespaces between the `-` operator and a value. Since DFM files are automatically generated, such whitespaces should not occur. Therefore, this cleaner approach is chosen, in favor of including negation on the parser level. 

## Conclusion

That's it. Now we posess a usable DFM grammar which we can easily parse using **ANTLR4**. In the next sections we will test out our grammar with examples provided.
