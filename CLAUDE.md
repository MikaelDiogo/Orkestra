# CLAUDE.md — Orkestra

Guia rápido para agentes de IA neste repositório. As regras completas estão em [`CONSTITUITION.md`](CONSTITUITION.md); **em caso de conflito, a Constitution vence**.

## O projeto

O **Orkestra** organiza o trabalho do time de vendas em um ambiente **simulado e autocontido**:

- **Vendedor:** organiza a própria agenda (calendário) e registra as atividades com cada lead (ligação, e-mail, reunião etc.).
- **Diretor:** acompanha as atividades dos vendedores dele e vê o que foi feito e o que está pendente.
- **CRM próprio:** leads, negócios e atividades ficam no banco do próprio projeto. Não existe CRM externo.

**Nada é integrado nem pago.** Não há Bitrix, Microsoft 365, LinkedIn, WhatsApp, telefonia nem LLM. Contato com lead é apenas **registrado** no sistema e nunca enviado de verdade.

Projeto de disciplina feito em dupla, com Spec-Driven Development:

- Mikael Diogo Ferreira — `@MikaelDiogo`
- Davi Marques — `@dev-davimarques`

## Estado atual

- Ainda **não há código**, só a Constitution e os relatórios em `relatorios/`.
- O escopo foi reduzido a um MVP depois do parecer do instrutor (`relatorios/relatorio_1.md`). Há **uma empresa só** (implícita) e **dois papéis**: Diretor e Vendedor.
- Não implemente nada antes de existir spec aprovada em `specs/`.

## Stack

- Backend: Java 21, Spring Boot 3, PostgreSQL + Flyway
- Frontend: Vite + React + TypeScript (`strict`, sem `any`), Mantine
- Testes: JUnit 5, Testcontainers, ArchUnit, Vitest

Arquitetura hexagonal (`domain` não importa infraestrutura/Spring/JPA) e autorização sempre no backend.

## Fluxo de trabalho

```text
specs/NNN-nome/spec.md → aprovação humana → plan.md → tasks.md → implementação → PR → revisão → merge (squash)
```

- Branches: `feat/<tema>` (humano), `agent/<tema>` (agente), `fix/<tema>`.
- Commits em Conventional Commits (`feat:`, `fix:`, `test:`, `docs:`, `refactor:`).
- Quem escreve não aprova: o autor nunca aprova nem faz merge do próprio PR.

## Regras que o agente não pode quebrar

1. **Nunca** fazer push/merge na `main`, aprovar PR, nem alterar a Constitution.
2. **Nunca** alterar requisitos, critérios de aceitação ou o `status` de uma spec. Se a spec estiver errada ou faltar informação, **pare e pergunte**.
3. **Sem integrações externas:** não adicionar chamada de rede para serviço de terceiros nem dependência paga. Tudo é simulado dentro do projeto.
4. **Só dados sintéticos** em código, testes, fixtures, seeds e logs (`lead.teste@exemplo.invalid`). Segredos ficam em variáveis de ambiente.
5. **Isolamento por papel** no backend, com teste: o Vendedor só vê o que é dele, e o Diretor só vê os vendedores dele.
6. Migrações, segurança, dependências novas, regras de domínio, CI/Docker e diffs com mais de 400 linhas: o agente só **propõe** em PR, e um humano revisa.

Detalhes e a Definição de Pronto: seções 4, 6 e 7 da [Constitution](CONSTITUITION.md).
