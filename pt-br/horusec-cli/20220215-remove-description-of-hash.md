# Remover descrição na geração do hash

- **Status:** proposto
- **Decisores:** [wiliansilvazup](https://github.com/wiliansilvazup), [nathanmartinszup](https://github.com/nathanmartinszup), [matheusalcantarazup](https://github.com/matheusalcantarazup), [iancardosozup](https://github.com/iancardosozup)
- **Date:** 2022-02-15
- **Tags:** technology, rules, semantic, security, engine, sast, horusec

## Contexto e Problema

Horusec está recebendo várias contribuições no projeto, visando uma melhoria no projeto atualmente precisamos mitigar os riscos se mantemos a descrição na geração do hash assim quando surgir uma melhoria na descrição da vulnerabilidade pode ser alterado sem se preocupar com os riscos de "breaking changes".

## Decisão

Iremos remover o campo `descrição` na geração dos hashes e iremos criar um campo novo chamado `old_hashes` na exportação da vulnerabilidade para que as ferramentas externas tenham opção de tratar qualquer um dos hashes antigos ou novos.

### Será preciso alterar o devkit para ter este novo compo de "OldHashes"
No projeto [Horusec-Devkit](https://github.com/ZupIT/horusec-devkit) temos a [struct de vulnerabilidade](https://github.com/ZupIT/horusec-devkit/blob/main/pkg/entities/vulnerability/vulnerability.go#L28) nela será preciso criar um novo campo como por exemplo:
```
type Vulnerability struct {
  ...
  OldHashes pq.StringArray `json:"oldHashes" gorm:"type:text[]"`
  ...
}
```

### Será preciso comunicar essa alteração a comunidade a partir que decidimos seguir com o desenvolvimento
A comunicação pode ser feita em todos os processos de desenvolvimento do horusec:
- Desenvolvimento: [cobertura para a ADR](https://github.com/ZupIT/horusec/issues/990)
- BugHunting: [validação da issue](https://github.com/ZupIT/horusec/issues/990)
- Beta: adicionando em nossa release notes e [atualizando a issue](https://github.com/ZupIT/horusec/issues/990)
- Release Candidate: adicionando em nossa release notes e [atualizando a issue](https://github.com/ZupIT/horusec/issues/990)

### Será preciso alterar no Horusec-Platform para suportar o campos de hash antigo e hash novo
No Horusec-Platform devemos validar quando receber este campo `OldHashes` e verificar se ele já existe na nossa base de dados.
Caso exista qualquer hash, então deveremos utilizar o que já estáva cadastrado para a análise atual.
Caso não exista qualquer hash, então deveremos cadastrar este novo.

### Será preciso adicionar um aviso na Horusec-CLI
Na Horusec-CLI deveremos verificar se alguma vulnerabilidade está utilizando ambos hashes antigos e novos.
Caso o usuário esteja exportando sua análise para outra origem diferente do tipo texto (json, sonarqube, sarif, horusec-api) deveremos adicionar um aviso na saída da aplicação demonstrando para o usuário que teve esta alteração e ele deverá ficar atento as mudanças.

### Após trocar para um novo formato deve-se aguardar pelo menos três versões minors ou seis meses para alterar novamente
Não queremos manter código que não faça sentido para o projeto, pois isto é um "Workaround" para ser transparente com a comunidade e que estamos preocupado com a experiencia de usuário no projeto.
Para que seja possível remover o campo `OldHashes` deve-se esperar pelo menos três versões minors ou seja 6 meses.

## Opções consideradas

### Apenas remover a descrição da geração do hash e deixar quebrar nas exportações
#### Prós
  - Neste cenário seria mais simples as mudanças e o impacto no código bem menor.
#### Contras
  - A experiencia de usuário seria bem ruim caso exista uma aplicação utilizando o projeto com vários tratamentos de falso positivos e riscos aceitos.

### Manter o formato antigo de forma estática
#### Prós
  - Seria interessante também criar um arquivo estático `old_descriptions` e adicionar ele no projeto para que possamos gerar o hash baseado nele.
#### Contras
  - A manutenção disto seria bastante grande e estamos apenas empurrando o problema para um período de curto prazo.

### Deixar o campo descrição e criar um novo campo "new_description"
#### Prós
  - Assim como remover a descrição neste formato também seria interessante e o impacto no código bem menor.
#### Contras
  - Porém caímos no mesmo cenário onde a manutenção seria bem grande e ignoramos totalmente a experiencia de usuário da aplicação.

## Links