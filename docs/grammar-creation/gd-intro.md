# Designing the DFM Grammar

This documentation outlines the components and rules that need to be followed when desgning an ANTLR-specific grammar for Delphi DFM files.

## Foreword

The grammar creation exemplified in this documentation is specifically designed with the [4FOOD ERP](https://www.4foodsoftware.com) system in mind, property of [Estrategy](https://www.estrategysoftware.com). It follows, that this documentation is not a generalized tutorial for parsing Delphi DFM files, although many concepts apply.

Everything described in this document is specifically tailored to ANTLRv4. Using previous versions of ANTLR would result in several major issues.

This document assumes familiarity with general concepts of lexers, parsers, grammars, parsing algorithms and methods, and some ANTLR-specific syntax. Knowledge on ANTLR4 features is a prerequisite, such as its usage of the **LL(k)** parsing algorithm - refer to [Parsing Algorithms and LL(k)](https://en.wikipedia.org/wiki/LL_parser#:~:text=The%20LL(k)%20parser%20is,alphabet%20are%20finite%20in%20size.) for more information.

The examples provided throughout the document build upon each other. Therefore, it is necessary to understand them for the purpose of contextualizing and informing on all decisions taken.

In case further context is needed and the reader would like to delve deeper on the concepts and topics presented in this documentation, we highly recommend the following sources:

- [The Definitive ANTLR 4 Reference](https://pragprog.com/titles/tpantlr2/the-definitive-antlr-4-reference/)
- [Stanford University - CS143 Compilers](https://web.stanford.edu/class/archive/cs/cs143/cs143.1128/)
- ["The Dragon Book" - Compilers: Principles, Techniques, and Tools](https://repository.unikom.ac.id/48769/1/Compilers%20-%20Principles,%20Techniques,%20and%20Tools%20(2006).pdf)
- [Delphi Language Reference](https://docwiki.embarcadero.com/RADStudio/Athens/en/Delphi_Language_Reference)

### Abstract Language Components

This section describes the abstract components considered for designing any grammar.

- `Sequence`: of elements such as values in array.
- `Choice`: between multiple, alternative phrases (such as different kinds of statements in language).
- `Token dependence`.
- `Nested phrase`: self-similar language construct (ex. aithmentic expression or nested statement blocks in language).
