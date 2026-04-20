# Commencement

Previous versions of *ANTLR* possessed the capability of automatically generating *ASTs* from input files. However, in its latest version (that is version 4, at the time of writing this documentation), this automatic generation feature no longer exists. Fortunately, *ANTLRv4* provides the necessary tools to build our own *AST* generator. The rest of this chapter will walk you through all steps of the *AST* generator's implementation.

# Updating the Grammar

For implementing the *AST* generation, we will be leveraging the tree-walking feature of *ANTLR*. The first step will be the upgrade our DFM grammar with *ANTLR's* grammar annotation feature.

The upgraded grammar looks like this:

`../../DelphiDFM.g4`
```yaml
grammar DelphiDFM ;

file             : block+ EOF 
                 ;

// Parser Rules
block            : ('inherited' | 'object' | 'inline') ID ':' ID propertyBlock* block* 'end'   # beginBlock
                 ;

propertyBlock    : SUBPROPERTY '=' value                                                       # subProperty
                 | ID '=' value                                                                # valueProperty
                 ;

value            : INT                                                                         # intValue
                 | FLOAT                                                                       # floatValue
                 | STRING                                                                      # stringLiteralValue
                 | ID                                                                          # stringValue
                 | ARRAY                                                                       # arrayValue
                 | collectionItems                                                             # collectionItemValue
                 | parenList                                                                   # parenListValue
                 | SUBPROPERTY                                                                 # dottedIdValue
                 ;

collectionItems  : '<' item* '>' ;
item             : 'item' propertyBlock* 'end' ;

parenList        : '(' (INT | FLOAT | STRING)* ')' ;

// Lexer Rules
INHERITED        : 'inherited' ;
OBJECT           : 'object' ;
INLINE           : 'inline' ;
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

## Breaking Down the Changes

### Block Rule

Our first update involves the inclusion of *inline* as an alternative option for component declarations. It follows, that we needed to include it as a reserved keyword in our *Lexer Rules* as well.

### Value Rule Additions

First, we included the `parenList` rule, to account for properties such as `DesignSize = (100 \n 200)`, where certain values can be included between parentheses, only separated by newlines.

The second addition, was the inclusion of our `SUBPROPERTY` rule as a value. The rationale behind it, is that certain values can be assigned to properties within DFM files. For instance, `OptionsImage.Images = dmMain.ilNormal24x24`.

### Annotations

This is the big one. *ANTLR* allows including values such as our `# beginBlock` within the grammar. These values are called annotations, and they tell *ANTLR* to create methods with the specified names, when generating the parser. These annotations represent the core of our *AST* generator, as we will override the methods generated for them, to generate *ASTs*. In the next section we'll see what happened when we regenerated the parser, and we'll delve into the *AST* generation development.