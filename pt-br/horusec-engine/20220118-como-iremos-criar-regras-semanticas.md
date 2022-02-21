# Como iremos criar regras na análise semântica

- **Status:** proposto
- **Decisores:** [wiliansilvazup](https://github.com/wiliansilvazup), [nathanmartinszup](https://github.com/nathanmartinszup), [matheusalcantarazup](https://github.com/matheusalcantarazup), [iancardosozup](https://github.com/iancardosozup), [oliveirafelipezup](https://github.com/oliveirafelipezup)
- **Date:** 2022-01-18
- **Tags:** technology, rules, semantic, security, engine, sast

## Contexto e Problema

Nós queremos criar um novo motor usando a análise semântica do [tree-sitter](https://github.com/tree-sitter/tree-sitter) no horusec-engine para diminuir a quantidade de falsos positivos. Tendo esse contexto, como iremos criar as regras?


## Decisão

Escrever as regras usando código Go com base em AST/IR.

## Motivadores da decisão

1. **Por que usar GoLang para criar as regras?**
- Como o time é especializado em GoLang e somos um time pequeno não vale a pena manter um fork de cada linguagem do tree-sitter.
- O Horusec terá um fork do [go-tree-sitter](https://github.com/smacker/go-tree-sitter), pois é mantido por uma pessoa não por uma empresa.
- A técnica utilizada por ele é o [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface).
- Esse pacote `go-tree-sitter` implementa binds de código `C` para `GoLang`. Onde o código `C` representa o parse da linguagem como, por exemplo:
  - [tree-sitter-go](https://github.com/tree-sitter/tree-sitter-go)
  - [tree-sitter-java](https://github.com/tree-sitter/tree-sitter-java)
  - [tree-sitter-javascript](https://github.com/tree-sitter/tree-sitter-javascript)
  - etc...

2. **Para continuar no formato Open Core da análise semântica as regras podem ser implementadas externamente:**
- Neste formato, conseguimos usar o metodo de `plugins` do Golang e rodar códigos em runtime.
- Teremos um desafio, em que os plugins devem ser compilados com a mesma versão de Go da CLI.
- Poderemos ter mais um comando chamado `horusec install` e o usuário passará o repositório com as regras externas que serão necessárias para serem adicionadas ao motor de análise semântica
- Poderemos ter uma lista de plugins na manager e o horusec identifica e instala em sua base antes de iniciar a análise.

## Opções consideradas

### Utilizar a propria linguagem de programação como linguagem de regras

#### Prós
  - Usuários não precisam aprender nada "novo" para escrever suas próprias regras
#### Contras
  - Precisaríamos manter um fork de cada grammar do tree-sitter (considerando que teríamos sintaxes customizadas, como metavariables $name, e ellipis ...)
  - Talvez seja difícil analisar contextos fora de uma função (por exemplo, uma função que processa um argumento (lê um arquivo) pode ser considerado vulnerável, porém se quem chama essa função só utiliza argumentos hard coded então ela não é vulnerável)
  - Fazer matching de código considerando contexto é difícil
  - Semgrep já faz isso, o que traríamos de novo/diferente?
  - Somos uma ferramenta de segurança, uma implementação de regras como essa seria genérica, de uso geral, para buscar por padrões de código (pode ser vulnerabilidade, code smell, bugs, etc), vale a pena o esforço?

## Links
- [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface)
- [tree-sitter](https://github.com/tree-sitter/tree-sitter)
- [GopherCon 2017: Keith Randall - Generating Better Machine Code with SSA](https://www.youtube.com/watch?v=uTMvKVma5ms&list=PL29r0-cYa9MrA_X9SDwxSRJrFrifLeQ34&index=4)
- [go-tree-sitter](https://github.com/smacker/go-tree-sitter)
