# CLAUDE.md — Orkestra

Guia rápido para agentes de IA neste repositório. As regras completas estão em [`CONSTITUITION.md`](CONSTITUITION.md); **em caso de conflito, a Constitution vence**.

## O projeto

O **Orkestra (Orkestrador)** fica entre a prospecção e o time de vendas: lê negócios no CRM (Bitrix24), valida o pacote de outreach, dispara uma **cadência de 15 dias** (e-mail, LinkedIn, ligação, WhatsApp) e audita a execução com evidência e feedback. É multi-tenant e lida com dados pessoais de leads (LGPD).

Projeto de disciplina feito em dupla, com Spec-Driven Development:

- Mikael Diogo Ferreira — `@MikaelDiogo`
- Davi Marques — `@dev-davimarques`

## Estado atual

- Ainda **não há código**, só a Constitution e os relatórios em `relatorios/`.
- O `relatorios/relatorio_1.md` (parecer do instrutor) recomenda reduzir o escopo: **uma linguagem só** e **um fluxo só** (coordenador entrega a lista de contatos, vendedor marca o que concluiu), sem multi-tenant no MVP. **A dupla ainda vai decidir esse recorte.** Não implemente nada antes de existir spec aprovada que reflita essa decisão.

## Stack planejada (sujeita ao recorte acima)

- Backend: Java 21, Spring Boot 3, PostgreSQL + Flyway
- Frontend: Vite + React + TypeScript (`strict`, sem `any`), Mantine
- Testes: JUnit 5, Testcontainers, WireMock, ArchUnit, Vitest

Arquitetura hexagonal (`domain` não importa infraestrutura/Spring/JPA), integrações atrás de portas (`CrmPort` etc.), autorização sempre no backend, idempotência por `negócio + passo`, efeitos externos via outbox.

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
3. **Dados pessoais e segredos** (leads reais, credenciais) nunca entram em código, testes, fixtures, logs, commits ou prompts. Use dados sintéticos (`lead.teste@exemplo.invalid`).
4. **Nenhum efeito externo real**: toda automação fica atrás de `orkestrador.automation.<nome>.enabled`, com default `false`; testes e `dev` não fazem chamadas de rede reais.
5. **Isolamento de tenant e papel** no backend, com teste para cada endpoint ou consulta nova.
6. Migrações, segurança, dependências novas, integrações, regras de domínio, CI/Docker e diffs com mais de 400 linhas: o agente só **propõe** em PR, e um humano revisa.

Detalhes e a Definição de Pronto: seções 4, 6 e 7 da [Constitution](CONSTITUITION.md).
