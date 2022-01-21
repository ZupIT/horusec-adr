# How we will create rules in semantic analysis

- **Status:** proposed
- **Decisores:** [wiliansilvazup](https://github.com/wiliansilvazup), [nathanmartinszup](https://github.com/nathanmartinszup), [matheusalcantarazup](https://github.com/matheusalcantarazup), [iancardosozup](https://github.com/iancardosozup), [oliveirafelipezup](https://github.com/oliveirafelipezup)
- **Date:** 2022-01-18
- **Tags:** technology, rules, semantic, security, engine, sast

## Context and problem

We want to create a new engine using the semantic analysis of the [tree-sitter](https://github.com/tree-sitter/tree-sitter) In the Horusec-Engine to decrease the amount of false positives.Having this context, how will we create the rules?


## Decision

Write rules using GO code based on AST / go

## Decision-making motivators

1. **Why use Golang to create the rules?**
- As the team specializes in Golang and we are a small team is not worth keeping a fork of each language of the Tree-Sitter.
- Horusec will have a fork of the [go-tree-sitter](https://github.com/smacker/go-tree-sitter), For it is maintained by a person not by a company.
- The technique used by it is the [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface).
- This package `go-tree-sitter` Implements Binds of Code `C` for `GoLang`. Where the code `C` represents the parse of language for example:
  - [tree-sitter-go](https://github.com/tree-sitter/tree-sitter-go)
  - [tree-sitter-java](https://github.com/tree-sitter/tree-sitter-java)
  - [tree-sitter-javascript](https://github.com/tree-sitter/tree-sitter-javascript)
  - etc...

2. **To continue in the Open Core format of the semantic analysis the rules can be implemented externally:**
- In this format we managed to use the method of `plugins` from Golang and run codes in Runtime.
- We will have a challenge that plugins should be compiled with the same GO version of CLI.
- We can have another command called `horusec install` and the user passes the repository with the external rules that need to be added in the semantic analysis engine
- We can have a list of plugins in Manager and Horusec identifies and installs at your base before starting the analysis.

## Options considered

### Use the own programming language as a rule language

#### Pros
  - Users do not need to learn anything "new" to write their own rules
#### Cons.
  - We need to keep a fork of each grammar of the tree-sitter (Considering that we would have customized syntaxes, such as metavariables $name, and ellipis ...)
  - It may be difficult to analyze contexts out of a function (for example, a function that processes an argument (reads a file) can be considered vulnerable, but if who calls this function only uses hard coded arguments then it is not vulnerable)
  - Do code matching considering context is difficult
  - Semgrep already do this, what we track again / different?
  - We are a security tool, an implementation of rules like this would be general, of general use, to search for code standards (can be vulnerability, Code Smell, Bugs, etc), is it worth the effort?

## Links
- [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface)
- [tree-sitter](https://github.com/tree-sitter/tree-sitter)
- [GopherCon 2017: Keith Randall - Generating Better Machine Code with SSA](https://www.youtube.com/watch?v=uTMvKVma5ms&list=PL29r0-cYa9MrA_X9SDwxSRJrFrifLeQ34&index=4)
- [go-tree-sitter](https://github.com/smacker/go-tree-sitter)
