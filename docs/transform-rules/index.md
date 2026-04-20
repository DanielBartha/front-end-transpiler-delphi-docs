# Introduction

The design of transformation rules (and later on the transformation model) is regarded as the most complex component of this tool. It is a crucial step, which necessitates careful architecting to ensure a valid output.

The transformation rules are comprised of a series of design decisions that specify how the conversion from source *AST* to target *AST* should be executed. This encapsulates the core component of the **Transformation Model**, regarded as the **Semantic Transformation Layer** within the transpiler pipeline.

It is recognized that the design of node notations and transformation rules is an iterative process which is destined to evolve as the legacy codebase is transpiled. As such, transpiling legacy subsets beyond the scoped representative subset will necessitate the expansion of *AST* node notations and transformation rules. 

## Preparations

In order to transpile a DFM file to React, the first step is to identify all dependencies of the chosen file. Determining the inheritance hierarchy of a file is a crucial step in establishing the transformation sequence.

Given a 5-level deep inheritance hierarchy of a chosen subset, we shall establish the philosophy of designing transformations for DFM components. The chosen subset's hierarchy has been traced in the figure below, and is used as reference throughout the process of designing *AST* node notations and transformation rules:

![Class Diagram of ActionsFrame.dfm Tracing its Inheritance Hierarchy](assets/dfm-composition-diagram.png)

The transpiling strategy will follow a bottom-up approach, working upwards from the furthest leaf-node: `ActionsFrame.dfm`. This means that stubs will be created for missing functionality as the inheritance hierarchy gets transpiled sequentially.