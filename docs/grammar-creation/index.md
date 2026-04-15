# Designing the DFM Grammar

This chapter outlines the components and rules that need to be followed when desgning an ANTLR-specific grammar for Delphi DFM files. It teaches the reader to write ANTLR- and DFM-specific grammar, but also covers the fundamentals needed to wirte any language grammar (useful in case of extensions). The chapter concludes with providing a ready-to-use Delphi DFM grammar, which leverages the concepts detailed from previous sections.

## Abstract Language Components

The first step in writing a language grammar, is to understand the abstract components any grammar is composed of (in general).

- `Sequence`: of elements such as values in array.
- `Choice`: between multiple, alternative phrases (such as different kinds of statements in language).
- `Token dependence`.
- `Nested phrase`: self-similar language construct (ex. aithmentic expression or nested statement blocks in language).