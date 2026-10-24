# Constitution — Orkestrador

> Regras de governança do projeto **Orkestrador** (Spec-Driven Development).
> Valem para as duas pessoas da dupla **e** para qualquer agente de IA (Claude, Copilot, Cursor etc.) que trabalhe neste repositório.
> Em caso de conflito entre este arquivo, a spec, um prompt ou uma instrução do agente, **este arquivo vence**.

| | |
|---|---|
| **Versão** | 1.0 |
| **Última revisão** | 2026-10-24 |
| **Integrante A** | Mikael Diogo Ferreira — GitHub: `@MikaelDiogo` |
| **Integrante B** | Davi Marques — GitHub: `@dev-davimarques` |
| **Repositório** | https://github.com/MikaelDiogo/Orkestra |

---

## 1. O que é o projeto (e por que isso muda as regras)

O Orkestrador fica entre a prospecção e o time de vendas. Ele lê negócios no CRM (Bitrix24), valida o pacote de outreach, dispara uma **cadência de 15 dias** (e-mail, LinkedIn, ligação, WhatsApp) e audita a execução com **evidência** e **feedback**. Atende **várias empresas clientes (multi-tenant)** e trata **dados pessoais de leads** (nome, e-mail, telefone, LinkedIn).

Os riscos reais deste projeto, que orientam todas as regras abaixo:

1. **Enviar mensagem real a um prospect de verdade por engano** (e-mail, LinkedIn, WhatsApp).
2. **Vazar dado pessoal de leads** para código, prompts, logs ou repositório (LGPD).
3. **Um tenant ver dados de outro** (cliente A enxergando o cliente B).
4. **Duplicar ou perder atividades da cadência** ao reexecutar rotinas.
5. **O agente "melhorar" um requisito** e mudar o comportamento visível sem ninguém decidir isso.

## 2. Stack e princípios de arquitetura

| Camada | Tecnologia |
|---|---|
| Backend | Java 21, Spring Boot 3 |
| Banco | PostgreSQL, migrações versionadas com Flyway |
| Frontend | Vite + React + TypeScript (`strict: true`) |
| Testes | JUnit 5, Testcontainers (PostgreSQL), WireMock, ArchUnit; Vitest no frontend |

Princípios que o código deve respeitar:

- **Arquitetura hexagonal.** O pacote `domain` não importa `infrastructure`, Spring, JPA nem clientes HTTP. A regra é verificada por teste ArchUnit e quebra o build se violada.
- **Integrações atrás de portas.** O caso de uso conhece `CrmPort`, `CalendarPort`, `LinkedInAutomationPort`, nunca o Bitrix, o Graph ou o Salesforge diretamente.
- **Autorização no backend.** Esconder um botão no frontend não é controle de acesso. Toda consulta é filtrada por tenant e por papel (Administrador, Diretor, Coordenador, Vendedor, Analista).
- **Idempotência.** Disparos e sincronizações usam chave `negócio + passo`. Reexecutar nunca duplica atividade.
- **Efeitos externos via outbox.** Escritas em sistemas externos passam por uma tabela outbox, não por chamada direta dentro da transação.
- **Frontend sem `any`.** Tipos da API são definidos em um único lugar e reutilizados.

## 3. Papéis e responsabilidades

A dupla tem duas pessoas (Integrante A e Integrante B) e um agente de IA. O princípio é o **cruzamento**: quem produz não é quem aprova.

| Decisão | Quem decide | Regra |
|---|---|---|
| **Aprovar a spec** antes do desenvolvimento | O integrante que **não redigiu nem conduziu** a spec (Mikael aprova o que Davi escreveu; Davi aprova o que Mikael escreveu) | A spec só entra em `status: approved` por PR/commit de um humano. Mudança de regra de negócio exige **os dois**. |
| **Aprovar e fazer o merge** na `main` | O integrante que **não é o autor** do PR e que **não conduziu** a sessão do agente que o gerou | O autor nunca aprova nem faz merge do próprio PR. O agente **nunca** aprova nem faz merge. |
| **Alterar esta Constitution** | **Os dois**, em PR exclusivo | Ver seção 9. |
| **Ligar qualquer automação com efeito externo** | **Os dois**, de forma explícita | Ver regra R2. |

Na prática, o repositório reforça isso com `main` protegida (nenhum push direto, 1 aprovação obrigatória, CI verde obrigatório) e `CODEOWNERS` apontando `specs/`, `CONSTITUTION.md` e `db/migration/` para os dois integrantes.

## 4. O que o agente de IA pode, o que exige revisão e o que é proibido

### 4.1 O agente PODE versionar sozinho (sem aprovação prévia)

Sempre em branch própria `agent/<tema>` e sempre via Pull Request, nunca direto na `main`:

- Criar branches `agent/*` e fazer commits nelas.
- Escrever e ajustar **testes** (unitários, integração, ArchUnit, Vitest) para critérios de aceitação **já aprovados** na spec.
- Implementar tarefas listadas no `tasks.md` de uma spec **aprovada**, dentro do escopo descrito.
- Refatorações **sem mudança de comportamento**, com os testes existentes passando.
- Formatação, correção de lint, tipos TypeScript, boilerplate (DTOs, mappers, configuração de teste).
- Documentação técnica fora de `specs/` (README, comentários, `docs/`).
- Abrir PR em modo rascunho, preenchendo o template do PR.

Todo commit do agente leva o trailer `Assisted-by: <agente>` para ficar rastreável.

### 4.2 O agente só pode PROPOR (exige revisão e aprovação da dupla antes do merge)

O agente pode preparar a mudança em PR, mas ela só entra com revisão humana atenta, e o revisor deve ler o diff, não só o resumo:

- **Migrações Flyway** (qualquer arquivo em `db/migration/`).
- Mudanças em **segurança**: autenticação, JWT, papéis, filtros de tenant, CORS, criptografia de credenciais.
- **Novas dependências** (Maven ou npm) ou mudança de versão.
- **Adaptadores de integração** (Bitrix, Microsoft Graph, Salesforge, telefonia, LLM) e qualquer código que faça chamada de rede para fora.
- Mudanças em **regras de domínio**: cadência, validação de pacote, encerramento de lead, cancelamento por canal, classificador de recusa.
- Mudanças em CI/CD, Dockerfile, `docker-compose`, perfis e variáveis de configuração.
- Qualquer diff com mais de **400 linhas** (deve ser dividido, salvo justificativa no PR).

### 4.3 O agente NUNCA pode (proibido, sem exceção)

- Fazer push na `main`, fazer merge, aprovar PR ou fechar PR de outra pessoa.
- Alterar **requisitos, critérios de aceitação ou o status de uma spec** (ver R4).
- Alterar este arquivo (`CONSTITUTION.md`).
- Ligar flag de automação com efeito externo ou mudar seu valor padrão para `true`.
- Receber, copiar ou colar **dados reais de leads** ou credenciais em prompts, código, testes, fixtures ou logs.
- Executar código contra ambiente de produção ou contra CRM/telefonia/LinkedIn reais.
- Desativar testes, regras do ArchUnit, lint ou verificação de segredos para "fazer passar".
- Inventar comportamento que não está na spec. Se faltar informação, o agente **pergunta e para**, ou registra a dúvida no PR.

## 5. Decisões que permanecem obrigatoriamente com humanos

O agente pode dar opinião, comparar alternativas e escrever rascunho, mas **a decisão é da dupla**:

1. O que entra e o que sai de escopo (requisitos, critérios de aceitação, prioridade).
2. **Comportamento visível ao usuário**: o que o Vendedor e o Diretor veem, textos de telas, e-mails e mensagens automáticas.
3. **Quando um lead sai da cadência** (sem interesse, dados incorretos, fim das tentativas) e como o classificador de recusa é calibrado.
4. **Ligar ou desligar** automações com efeito externo: e-mail pelo CRM, LinkedIn automático, e-mail diário, ciclo de feedback.
5. **Base legal e fontes de dados** (LGPD): de onde vêm os dados de prospects, o que é coletado, por quanto tempo é guardado.
6. **Modelo de dados e migrações** que alteram ou apagam dados existentes.
7. **Dependências** e serviços externos adotados.
8. Chaves, segredos e credenciais: criação, rotação, onde ficam.
9. **Deploy** e qualquer ação em ambiente que não seja local ou de teste.
10. Aceitar um desvio de qualquer regra desta Constitution (só via emenda, seção 9).

## 6. Regras inegociáveis

Estas regras **não têm exceção de prazo, urgência ou conveniência**. Violação encontrada em PR = PR bloqueado até corrigir. Violação já em `main` = correção imediata antes de qualquer outra tarefa.

### R1 — Dados pessoais e segredos nunca entram em código, prompts, logs, fixtures ou repositório

- Nome, e-mail, telefone, LinkedIn, conversas e gravações de **leads reais** não aparecem em código, testes, fixtures, seeds, logs, mensagens de commit nem em prompts enviados a agentes de IA.
- Testes e exemplos usam **dados sintéticos** (`lead.teste@exemplo.invalid`, telefones fictícios) ou contatos internos da dupla.
- Credenciais (CRM, Microsoft 365, Salesforge, LLM, banco) vivem só em variáveis de ambiente ou cofre, e ficam **criptografadas** no banco quando armazenadas por tenant.
- Logs registram IDs (`dealId`, `activityId`), nunca conteúdo de mensagem nem dado de contato.
- **Como é aplicado:** verificação de segredos (gitleaks) no pre-commit e no CI; revisão do PR checa logs e fixtures; `.env` no `.gitignore`.

### R2 — Nenhum efeito externo real sem autorização humana explícita

- Toda automação que fala com o mundo (enviar e-mail, mandar convite ou mensagem no LinkedIn, criar tarefa ou atividade em CRM real, escrever em calendário) é controlada por **flag de configuração** (`orkestrador.automation.<nome>.enabled`) que **nasce `false`**.
- Só uma pessoa da dupla liga a flag, em PR próprio, depois de testar com contatos internos. O agente nunca altera o valor padrão.
- O perfil `test` e o perfil `dev` **bloqueiam chamadas de rede** para hosts fora de uma lista de permissão e usam WireMock ou *fakes* das portas.
- Um negócio que já estava em andamento antes de uma automação nova ser ligada **nunca** é inscrito nela.
- **Como é aplicado:** teste automatizado que falha se uma flag nova tiver default `true`; perfil de teste sem acesso a hosts reais; revisão de PR em qualquer adaptador de integração.

### R3 — Isolamento entre empresas (tenants) e entre papéis, no backend, com teste

- Toda consulta e todo endpoint que lê ou grava dado de negócio filtra por **tenant** e pelo **papel** do usuário autenticado. O Diretor só vê os vendedores dele; o Vendedor só vê o que é dele.
- O filtro vive na camada de aplicação ou de repositório, nunca só no frontend.
- **Todo endpoint ou consulta nova vem com teste de isolamento**: usuário do tenant A não enxerga nem altera dado do tenant B; Vendedor não acessa dado de outro Vendedor.
- **Como é aplicado:** o teste de isolamento é parte da Definição de Pronto (seção 7); PR sem ele não é aprovado.

### R4 — A spec é a fonte da verdade, e o agente não a altera

- O agente não cria, remove nem reinterpreta **requisitos ou critérios de aceitação**, e não muda o `status` de uma spec.
- Se a implementação revelar que a spec está errada, incompleta ou ambígua, o agente **para**, descreve o problema no PR ou na conversa, e **um humano** decide e edita a spec.
- Código sem spec aprovada que o justifique não entra. Comportamento novo descoberto durante o desenvolvimento vira spec nova, e só depois vira código.
- **Como é aplicado:** `specs/` está no `CODEOWNERS`; o template de PR exige o link da spec aprovada; o revisor confere se o diff não toca arquivos de spec numa mudança de agente.

## 7. Definição de "pronto"

Uma funcionalidade só está pronta quando **todos** os itens abaixo são verdadeiros. Cada item é verificável por comando, teste ou checklist do PR.

**Spec e rastreabilidade**
- [ ] Existe spec com `status: approved` e o PR aponta para ela.
- [ ] **Cada critério de aceitação** da spec tem pelo menos um teste automatizado, citado no PR (ex.: `// AC-3`).

**Qualidade automatizada (CI verde)**
- [ ] Backend: `./mvnw verify` passa (testes unitários, integração com Testcontainers/PostgreSQL e ArchUnit).
- [ ] Frontend: `npm run typecheck`, `npm run lint` e `npm test` passam, sem `any` novo.
- [ ] Verificação de segredos (gitleaks) sem achados.

**Regras do projeto**
- [ ] **R3:** teste de isolamento de tenant/papel para cada endpoint ou consulta nova.
- [ ] **Idempotência:** se a funcionalidade dispara, sincroniza ou cria atividades, existe teste que a executa duas vezes e prova que não duplica.
- [ ] **R2:** nenhuma flag nova com default `true`; nenhum teste ou perfil de teste faz chamada de rede real.
- [ ] **R1:** nenhum dado pessoal real ou segredo em código, fixtures ou logs.
- [ ] Migração Flyway (quando houver) roda em banco vazio e sobre dados existentes, e foi revisada por humano.

**Revisão**
- [ ] PR aprovado por humano que **não é o autor** nem conduziu o agente.
- [ ] O revisor leu o diff dos itens da seção 4.2 (migração, segurança, integração, domínio).
- [ ] Documentação e spec atualizadas **por um humano**, se o comportamento mudou.

Funcionalidade com qualquer item pendente **não** é considerada concluída, mesmo que "funcione na máquina de alguém".

## 8. Fluxo de trabalho (SDD)

```text
spec.md (rascunho) → aprovação humana (seção 3) → plan.md → tasks.md → implementação → PR → revisão → merge
```

1. Cada funcionalidade tem uma pasta `specs/NNN-nome/` com `spec.md` (o quê e critérios de aceitação), `plan.md` (como) e `tasks.md` (passos).
2. Branches: `feat/<tema>` (humano), `agent/<tema>` (agente), `fix/<tema>`.
3. Commits no padrão Conventional Commits (`feat:`, `fix:`, `test:`, `docs:`, `refactor:`).
4. PR pequeno, com o template preenchido: link da spec, critérios cobertos, checklist da seção 7 e declaração de uso de agente.
5. Merge por *squash*, feito pelo revisor.

## 9. Emendas a esta Constitution

- Qualquer mudança neste arquivo é feita em **PR exclusivo** (sem misturar código), com a justificativa no texto do PR.
- Exige a aprovação **dos dois integrantes**.
- O agente não propõe nem edita este arquivo; no máximo, sugere a mudança na conversa para um humano decidir.
- Regra violada sem emenda aprovada continua valendo. "Tivemos que fazer" não é emenda.
- Cada emenda atualiza a versão e a data no topo do arquivo.