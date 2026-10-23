# Relatório — Orkestra

## Viabilidade

- Grupo: Dupla 02
- Integrantes: Davi Marques e Mikael Diogo Ferreira
- Repositório: https://github.com/MikaelDiogo/Orkestra
- Stack: TypeScript, React, Vite, Mantine e Java com Spring Boot.
- Veredito: Acima do prazo

Como está desenhado, não cabe na disciplina. São três papéis, vários tenants, pacotes de outreach, agenda semanal, feedback e métrica de produtividade, em duas linguagens. 

## SWOT

### Forças

O problema de distribuição de leads é concreto. Vendedor, diretor e administrador estão nomeados, e a dupla percebe que o acesso não pode ser o mesmo para todos.

### Fraquezas

Multitenancy, RBAC e dois backends são o produto inteiro, não um recorte de semana. A justificativa cobre só Spring e não explica React, Vite nem Mantine.

### Oportunidades

Um tenant implícito, dois papéis e um fluxo: o coordenador entrega uma lista de contatos e o vendedor marca o que fez. Isolamento entre empresas e pacotes de outreach saem do MVP.

### Ameaças

Sem esse corte, a constitution nasce grande demais e a implementação não chega a um fluxo demonstrável. Erro de isolamento entre tenants vira o assunto do projeto.

## Stack técnica

**Parecer: Inadequada no conjunto.** Duas linguagens não cabem nesta etapa; a stack recomendada é TypeScript de ponta a ponta, com React e uma API na mesma linguagem, e um banco só.

React, Vite e Mantine formam uma interface adequada. Java com Spring Boot também seria uma API adequada. Juntas, as duas viram dois sistemas para especificar, subir e integrar, e a justificativa só explica o Spring, por causa de multitenancy e RBAC. Com o recorte de uma empresa só e dois papéis, esse motivo deixa de sustentar a segunda linguagem.

A stack recomendada é TypeScript no React com Vite e Mantine, mais uma API pequena na mesma linguagem, com SQLite ou PostgreSQL. A dupla mantém a tela que já escolheu e elimina o salto entre ecossistemas. Se a preferência for Java, o caminho inverso também serve: Spring Boot, H2 e uma tela mínima, sem React nesta etapa. Os dois caminhos ao mesmo tempo não.

## Recomendação

**Reduzam o Orkestra a uma linguagem e a um fluxo: o coordenador entrega a lista de contatos e o vendedor marca o que concluiu.**

O Orkestra descreve um produto de vendas inteiro, e é isso que o torna inviável no calendário atual. Há três papéis, várias empresas no mesmo sistema, pacotes de outreach, agenda semanal e métrica de produtividade, além de duas linguagens ao mesmo tempo. A constitution precisa escolher um único caminho técnico, TypeScript do React até a API ou só Java com Spring Boot, e um único fluxo de trabalho. Esse fluxo pode ser simples: o coordenador monta a lista de contatos da semana e o vendedor marca o que já fez. A empresa é uma só, implícita, então ainda não há isolamento entre tenants. Outreach, auditoria genérica e painel de produtividade saem do texto desta etapa. Sem esse corte, a spec nasce maior do que a implementação consegue alcançar. A descrição do GitHub, que fala em orquestração e auditoria, também precisa passar a contar a mesma história do PDF recortado, senão a dupla especifica um sistema e o repositório anuncia outro.
