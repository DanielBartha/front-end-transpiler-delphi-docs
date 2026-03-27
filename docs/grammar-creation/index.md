# Designing the DFM Grammar

This chapter outlines the components and rules that need to be followed when desgning an ANTLR-specific grammar for Delphi DFM files. It teaches the reader to write ANTLR- and DFM-specific grammar, but also covers the fundamentals needed to wirte any language grammar (useful in case of extensions). The chapter concludes with providing a ready-to-use Delphi DFM grammar, which leverages the concepts detailed in previous sections.

## Foreword

Everything described in this documentation overall, is specifically tailored to **ANTLRv4**. Using previous versions of ANTLR would result in several major issues.

The grammar creation exemplified in this chapter is specifically designed with the [4FOOD ERP](https://www.4foodsoftware.com) system in mind, property of [Estrategy](https://www.estrategysoftware.com). Although most concepts regarding grammar creation truthfully appply, this documentation and all examples provided are specifically designed with DFM files in mind.

This document assumes familiarity with the general concept of lexers, parsers, grammars, parsing algorithms and methods, and some ANTLR-specific syntax. Knowledge on *ANTLR4* features is not necessarily a prerequisite, as the essential ones are detailed throughout this documentation. However, we strongly recommend taking an overview on *ANTLR's* full feature list (check for sources below).

*ANTLRv4* relies on the **LL(k)** parsing algorithm, which is a highly suitable for parsing DFM files. This documentation does not delve into such levels of granularity as parsing algorithms. If the reader wishes to understand *ANTLR's* inner workings, which also substantiate its usage for DFM parsing, we recommend the sources at the end of this section.

The examples provided throughout the document build upon each other. Therefore, it is necessary to understand them for the purpose of contextualizing and informing on all decisions taken.

### Sources

In case further context is needed, and the reader would like to delve deeper on the concepts and topics presented throughout this documentation, we highly recommend the following sources:

- [ANTLR Main Page](https://www.antlr.org/index.html)
- [ANTLR4 GitHub Documentation](https://github.com/antlr/antlr4/blob/master/doc/index.md)
- [The Definitive ANTLR 4 Reference](https://pragprog.com/titles/tpantlr2/the-definitive-antlr-4-reference/)
- [Stanford University - CS143 Compilers](https://web.stanford.edu/class/archive/cs/cs143/cs143.1128/)
- ["The Dragon Book" - Compilers: Principles, Techniques, and Tools](https://repository.unikom.ac.id/48769/1/Compilers%20-%20Principles,%20Techniques,%20and%20Tools%20(2006).pdf)
- [Parsing Algorithms and LL(k)](https://en.wikipedia.org/wiki/LL_parser#:~:text=The%20LL(k)%20parser%20is,alphabet%20are%20finite%20in%20size.)
- [Delphi Language Reference](https://docwiki.embarcadero.com/RADStudio/Athens/en/Delphi_Language_Reference)
