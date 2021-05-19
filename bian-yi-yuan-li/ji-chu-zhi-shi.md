# 基础知识

编译原理概述

**A compiler translates the code written in one language to some other language without changing the meaning of the program.**

It is also expected that a compiler should make the target code efficient and optimized in terms of time and space.

常用概念

* Preprocessor

* Interpreter

  An interpreter, like a compiler, translates high-level language into low-level machine language.

  > Interpreter vs compiler
  >
  > Interpreter: reads a statement from the input, converts it to an intermediate code, executes it, then takes the next statment in sequence.
  >
  > Compiler: read the whole source code at once, creates tokens, checks semantics, generates intermediate code, executes the whole program and may invoke many passes.

* Assembler

* Linker

* Loader

* Cross-compiler

* Source-to-source Compiler

A compiler can broadly be devided into two phases based on the way they compile.

**Front-end Back-end**

The compilation process is a sequence of various phases. Each phase takes input from its previous stage, has its own representation of source program, and feeds its output to the next phase of the compiler.

* Lexical Analysis: `keywords`,`constants`, `identifiers`, `strings`, `numbers`, `operators`, `punctuations symbols` can be considered as tokens.

  ```
  <token-name, attribute-valute>
  
  // for example, in C 
  int value = 100;
  // convert to
  int (keyword), value (identifier), = (operator), 100 (constant) and ; (symbol)
  ```

* Syntax Analysis

* Semantic Analysis

  ```
  The semantic analyzer produces an annotated syntax tree as an output. 
  ```

* intermediate Code Generation

  ```
  The intermediate code should be generated in such a way that it takes it to be translated into the target machine code.
  ```

* Code Optimizatin

* Code Generatin

* Symbol Table

