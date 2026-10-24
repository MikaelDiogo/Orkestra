# Constitution — Orkestra

> Regras de governança do projeto **Orkestra** (Spec-Driven Development).
> Valem para as duas pessoas da dupla **e** para qualquer agente de IA (Claude, Copilot, Cursor etc.) que trabalhe neste repositório.
> Em caso de conflito entre este arquivo, a spec, um prompt ou uma instrução do agente, **este arquivo vence**.

| | |
|---|---|
| **Versão** | 1.1 (recorte do MVP) |
| **Última revisão** | 2026-10-24 |
| **Integrante A** | Mikael Diogo Ferreira — GitHub: `@MikaelDiogo` |
| **Integrante B** | Davi Marques — GitHub: `@dev-davimarques` |
| **Repositório** | https://github.com/MikaelDiogo/Orkestra |

---

## 1. O que é o projeto (MVP)

O Orkestra organiza o trabalho de um time de vendas em um **ambiente simulado e autocontido**:

- O **Vendedor** organiza a própria agenda (calendário) e registra as atividades feitas com cada lead (ligação, e-mail, reunião etc.).
- O **Diretor** acompanha as atividades dos seus vendedores: o que foi feito, o que está pendente e o que atrasou.
- O **CRM é do próprio projeto**: leads, atividades e agenda ficam no banco da aplicação.

**Fora do MVP:** CRM externo (Bitrix24 ou qualquer outro), integrações com Microsoft 365, LinkedIn, WhatsApp, telefonia ou LLM, qualquer serviço pago, várias empresas no mesmo sistema (há **uma empresa só**, implícita) e envio real de mensagens. Contato com lead é apenas **registrado**, nunca enviado.

Os riscos reais deste MVP, que orientam as regras abaixo:

1. **Um vendedor ver ou alterar dado de outro vendedor**, ou um Diretor ver vendedores que não são dele.
2. **Dado pessoal real ou segredo parar no repositório** (código, fixtures, logs).
3. **O escopo crescer de novo** e o fluxo principal não ficar demonstrável a tempo.
4. **O agente "melhorar" um requisito** e mudar o comportamento visível sem ninguém decidir isso.

## 2. Stack e princípios de arquitetura

| Camada | Tecnologia |
|---|---|
| Backend | Java 21, Spring Boot 3 |
| Banco | PostgreSQL, migrações versionadas com Flyway |
| Frontend | Vite + React + TypeScript (`strict: true`) + Mantine |
| Testes | JUnit 5, Testcontainers (PostgreSQL), ArchUnit; Vitest no frontend |

Princípios que o código deve respeitar:

- **Arquitetura hexagonal.** O pacote `domain` não importa `infrastructure`, Spring, JPA nem clientes HTTP. A regra é verificada por teste ArchUnit e quebra o build se violada.
- **Autocontido.** O sistema não chama serviços de terceiros. Tudo que seria integração é simulado dentro do projeto.
- **Autorização no backend.** Esconder um botão no frontend não é controle de acesso. Toda consulta é filtrada pelo papel (Diretor, Vendedor) e pelo dono do dado.
- **Frontend sem `any`.** Tipos da API são definidos em um único lugar e reutilizados.

## 3. Papéis e responsabilidades

A dupla tem duas pessoas (Integrante A e Integrante B) e um agente de IA. O princípio é o **cruzamento**: quem produz não é quem aprova.

| Decisão | Quem decide | Regra |
|---|---|---|
| **Aprovar a spec** antes do desenvolvimento | O integrante que **não redigiu** a spec | A spec só entra em `status: approved` por PR/commit de um humano. Mudança de regra de negócio exige **os dois**. |
| **Aprovar e fazer o merge** na `main` | O integrante que **não é o autor** do PR e que **não conduziu** a sessão do agente que o gerou | O autor nunca aprova nem faz merge do próprio PR. O agente **nunca** aprova nem faz merge. |
| **Alterar esta Constitution** | **Os dois** | Ver seção 9. |

Na prática, o repositório reforça isso com `main` protegida (nenhum push direto, 1 aprovação obrigatória, CI verde obrigatório) e `CODEOWNERS` apontando `specs/`, `CONSTITUITION.md` e `db/migration/` para os dois integrantes.

## 4. O que o agente de IA pode, o que exige revisão e o que é proibido

### 4.1 O agente PODE versionar sozinho (sem aprovação prévia)

Sempre em branch própria `agent/<tema>` e sempre via Pull Request, nunca direto na `main`:

- Escrever e ajustar **testes** para critérios de aceitação **já aprovados** na spec.
- Implementar tarefas listadas no `tasks.md` de uma spec **aprovada**, dentro do escopo descrito.
- Refatorações **sem mudança de comportamento**, com os testes existentes passando.
- Formatação, correção de lint, tipos TypeScript, boilerplate (DTOs, mappers, configuração de teste).
- Documentação técnica fora de `specs/` (README, comentários, `docs/`).

### 4.2 O agente só pode PROPOR (exige revisão e aprovação da dupla antes do merge)

- **Migrações Flyway** (qualquer arquivo em `db/migration/`).
- Mudanças em **segurança**: autenticação, papéis, filtros de acesso, CORS.
- **Novas dependências** (Maven ou npm) ou mudança de versão.
- Mudanças em **regras de domínio**: agenda, status de atividade, o que o Diretor enxerga.
- Mudanças em CI/CD, Dockerfile, `docker-compose`, perfis e variáveis de configuração.
- Qualquer diff com mais de **400 linhas** (deve ser dividido, salvo justificativa no PR).

### 4.3 O agente NUNCA pode (proibido, sem exceção)

- Fazer push na `main`, fazer merge, aprovar PR ou fechar PR de outra pessoa.
- Alterar **requisitos, critérios de aceitação ou o status de uma spec** (ver R4).
- Alterar este arquivo por conta própria.
- Adicionar integração com serviço externo ou dependência paga.
- Colocar dados pessoais reais ou credenciais em código, testes, fixtures ou logs.
- Desativar testes, regras do ArchUnit ou lint para "fazer passar".
- Inventar comportamento que não está na spec. Se faltar informação, o agente **pergunta e para**, ou registra a dúvida no PR.

## 5. Decisões que permanecem obrigatoriamente com humanos

O agente pode dar opinião, comparar alternativas e escrever rascunho, mas **a decisão é da dupla**:

1. O que entra e o que sai de escopo (requisitos, critérios de aceitação, prioridade).
2. **Comportamento visível ao usuário**: o que o Vendedor e o Diretor veem e os textos das telas.
3. **Regras de acompanhamento**: quando uma atividade conta como atrasada e o que o Diretor pode ver ou alterar.
4. **Modelo de dados e migrações** que alteram ou apagam dados existentes.
5. **Dependências** adotadas.
6. Aceitar um desvio de qualquer regra desta Constitution (só via emenda, seção 9).

## 6. Regras inegociáveis

Violação encontrada em PR = PR bloqueado até corrigir. Violação já em `main` = correção imediata antes de qualquer outra tarefa.

### R1 — Só dados sintéticos; segredos fora do repositório

- Código, testes, fixtures, seeds e logs usam **dados sintéticos** (`lead.teste@exemplo.invalid`, telefones fictícios).
- Senhas e chaves vivem só em variáveis de ambiente. `.env` fica no `.gitignore`.
- Logs registram IDs, nunca dado de contato.

### R2 — Sem integrações externas no MVP

- O sistema não faz chamada de rede para serviços de terceiros e não depende de nada pago.
- Agenda, CRM e registro de atividades são implementados dentro do projeto.
- Integrar qualquer serviço externo exige emenda a esta Constitution (seção 9).

### R3 — Isolamento por papel, no backend, com teste

- O **Vendedor** só vê e altera os próprios leads, atividades e agenda.
- O **Diretor** só vê os vendedores ligados a ele.
- O filtro vive na camada de aplicação ou de repositório, nunca só no frontend.
- **Todo endpoint ou consulta nova vem com teste de isolamento** (ex.: Vendedor A não acessa atividade do Vendedor B).

### R4 — A spec é a fonte da verdade, e o agente não a altera

- O agente não cria, remove nem reinterpreta **requisitos ou critérios de aceitação**, e não muda o `status` de uma spec.
- Se a implementação revelar que a spec está errada, incompleta ou ambígua, o agente **para**, descreve o problema e **um humano** decide e edita a spec.
- Código sem spec aprovada que o justifique não entra.

## 7. Definição de "pronto"

Uma funcionalidade só está pronta quando **todos** os itens abaixo são verdadeiros:

- [ ] Existe spec com `status: approved` e o PR aponta para ela.
- [ ] **Cada critério de aceitação** da spec tem pelo menos um teste automatizado, citado no PR (ex.: `// AC-3`).
- [ ] Backend: `./mvnw verify` passa (unitários, integração com Testcontainers e ArchUnit).
- [ ] Frontend: `npm run typecheck`, `npm run lint` e `npm test` passam, sem `any` novo.
- [ ] **R3:** teste de isolamento por papel para cada endpoint ou consulta nova.
- [ ] **R1/R2:** nenhum dado real, segredo ou chamada externa.
- [ ] PR aprovado por humano que **não é o autor** nem conduziu o agente.

## 8. Fluxo de trabalho (SDD)

```text
spec.md (rascunho) → aprovação humana (seção 3) → plan.md → tasks.md → implementação → PR → revisão → merge
```

1. Cada funcionalidade tem uma pasta `specs/NNN-nome/` com `spec.md` (o quê e critérios de aceitação), `plan.md` (como) e `tasks.md` (passos).
2. Branches: `feat/<tema>` (humano), `agent/<tema>` (agente), `fix/<tema>`.
3. Commits no padrão Conventional Commits (`feat:`, `fix:`, `test:`, `docs:`, `refactor:`).
4. PR pequeno, com link da spec, critérios cobertos e checklist da seção 7.
5. Merge por *squash*, feito pelo revisor.

## 9. Emendas a esta Constitution

- Qualquer mudança neste arquivo é feita em commit ou PR exclusivo (sem misturar código), com a justificativa.
- Exige a aprovação **dos dois integrantes**.
- O agente não edita este arquivo por conta própria; só aplica uma mudança pedida explicitamente pela dupla.
- Cada emenda atualiza a versão e a data no topo do arquivo.
