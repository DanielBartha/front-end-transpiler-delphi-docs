# Delphi Front End Transpiler

This documentation delves into all aspects of the front end transpiler tool, used for translating legacy Delphi DFM files into moder React/TypeScript code. 

It starts with parsing Delphi VCL components, represented by the `.dfm` files. Going into details of designing the DFM grammar, which provide a solid foundation and inform the reader on techniques used for expanding said grammar (if necessary in the future).

Once the foundations have been established, a refined and ready-to-use DFM grammar will be looked at, and will be parsed using **ANTLR4** in later sections. The foundation of this project *is* **ANTLRv4**, therefore it's setup and usage are also elaborated upon, in great detail. 

The transpiler tool will leverage *Abstract Syntax Trees (AST)* of the legacy code, and translate them into modern intermediate models of the same level. For this, the documentation delves into implementing *AST* generation from the created DFM grammar (of previous sections), to ultimately be used as reference in generating a modern React/TypeScript *AST*.

In concordance with the *AST generation* chapter, sections elaborating transformation rules will assimilate into it, as essential design decisions.

**To be continued after further developments...**
