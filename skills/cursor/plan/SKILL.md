---
name: plan
description: >-
  Gera o planejamento técnico de uma atividade em Markdown no chat, com
  objetivo, estratégia, caminho escolhido, considerações de segurança, ordem de
  implementação, rollout, testes (após triagem), observabilidade e pendências;
  confronta decisões em aberto do Discovery/entrada antes do plano final. Use
  quando o usuário pedir plano técnico, planejamento de implementação, plano de
  entrega, ou quiser transformar uma demanda/discovery em plano de execução.
disable-model-invocation: true
---

# Skill: Plan

## Objetivo

Transformar uma demanda (e, quando disponível, um Discovery) em um plano técnico estruturado, entregue apenas no chat.

---

## Decisões de design

### Entrega apenas no chat

| Ação | Quando | Onde |
|------|--------|------|
| **Entregar Markdown** | Sempre | Resposta do agente |
| **Salvar em arquivo** | Nunca | — |
| **Versionar no Git** | Fora do escopo | — |

**Por quê:** o plano é um artefato de alinhamento na conversa; persistir em disco gera ruído e compete com issues/PRs.

### Discovery (opcional)

Não há pré-requisito formal com a skill `discovery`. Se houver Discovery na conversa ou em `docs/discovery/`, use-o como contexto. Caso contrário, planeje com o que o usuário passar.

### Modo Plan do Cursor

Esta skill é compatível com o modo Plan do Cursor. Entregue o Markdown na resposta; **não** crie arquivo. O artefato interno do Cursor (`CreatePlan`) é opcional e **separado** do documento gerado por esta skill — não misture os dois.

---

## Responsabilidades

- Compreender a demanda e o contexto disponível.
- Alinhar o caminho de implementação ao padrão do projeto (`QUICK_START_GUIDE`, quando existir).
- Detectar decisões em aberto no Discovery/entrada e confrontá-las com o usuário antes do plano final.
- Definir objetivo, estratégia, caminho escolhido e ordem de implementação.
- Definir considerações e controles de segurança necessários.
- Definir estratégia de rollout e observabilidade.
- Triar com o usuário se a atividade deve conter testes **antes** de preencher a estratégia de teste.
- Registrar pendências objetivas — em especial decisões em aberto não resolvidas.

---

## Restrições

Nunca:

- escrever código de produção;
- criar ADR (se surgir decisão arquitetural, sugerir a skill `adr-simplificado`);
- persistir o plano em disco;
- inventar padrão do projeto se o `QUICK_START_GUIDE` não existir;
- inventar a resposta da triagem de testes;
- inventar ou omitir controles de segurança sem validar com o usuário;
- entregar o plano final fechando ou omitindo decisões em aberto sem questionar o usuário.

Caso o contexto seja insuficiente para preencher as seções obrigatórias com segurança, **pare**, faça perguntas e liste o que falta.

---

## Fluxo

### Etapa 1 — Compreender o contexto

Leia a mensagem do usuário, Discovery (se houver), issues e documentos relevantes.

### Etapa 2 — Ler o padrão do projeto

Procure em `docs/` um arquivo chamado `QUICK_START_GUIDE` (aceite `.md` e variações de case, ex.: `docs/QUICK_START_GUIDE.md`).

- Se existir: leia e use como referência do padrão de implementação no **Caminho escolhido**.
- Se não existir: continue sem bloquear; registre em **Pendências** se a ausência for lacuna relevante.

### Etapa 3 — Decisões em aberto (gate)

**Antes** de entregar o plano final, detecte decisões em aberto no Discovery e na entrada do usuário. Fontes típicas:

- seção **Dúvidas** do Discovery;
- seção **Informações ausentes** do Discovery;
- seção **Considerações de segurança** do Discovery (lacunas ou itens sem decisão);
- status **Pronto para Planejamento? Não**;
- escolhas ou alternativas ainda não fechadas na mensagem do usuário.

Se **não** houver decisões em aberto, siga para a Etapa 4 sem gate extra.

Se houver alguma:

1. **Pare** — não entregue o plano final ainda.
2. Liste cada decisão de forma acionável (o que precisa ser decidido / por que importa).
3. **Pergunte** ao usuário — trate como item valioso de alinhamento, não como nota passiva.
4. Aguarde as respostas quando possível.
5. O que for resolvido entra no plano (estratégia, caminho, segurança, rollout, etc.).
6. O que **permanecer sem resolução** vai para **9. Pendências** de forma explícita, com formulação clara (ex.: `Decisão em aberto: …` ou `Pendência de alinhamento: …`).

### Etapa 4 — Triagem de testes (gate)

**Antes** de escrever a seção **Estratégia de teste**, pergunte ao usuário:

> Esta atividade / este projeto deve conter testes?

Não invente a resposta. Aguarde a resposta do usuário.

- **Sim:** preencha **Estratégia de teste** (tipos, escopo, critérios mínimos).
- **Não:** na seção, registre em uma frase objetiva que testes estão fora do escopo desta atividade.

### Etapa 5 — Entregar o plano

Preencha todas as seções da saída obrigatória e **entregue o Markdown na resposta**.

- Não crie arquivo.
- Não commit.
- No modo Plan do Cursor: mantenha a entrega só no chat; trate o artefato do Cursor como separado.
- Preencha **Considerações de segurança** com base no Discovery (se existir) e nas mudanças planejadas.
- Se houver dúvidas sobre requisitos de segurança não documentados no Discovery, pergunte ao usuário antes de finalizar o plano.
- Inclua em **Pendências** todas as decisões em aberto não resolvidas na Etapa 3.

---

## Saída obrigatória

Gere um documento Markdown contendo exatamente estas seções:

```markdown
# Plano: [Título da atividade]

## 1. Objetivo

[O que se pretende alcançar com esta entrega.]

## 2. Estratégia

[Abordagem geral para realizar a atividade.]

## 3. Caminho escolhido

[Caminho técnico escolhido e por quê.]

Referência de padrão: [caminho do QUICK_START_GUIDE usado, ou "não encontrado".]

## 4. Considerações de segurança

### Dados sensíveis
[Quais dados / Como proteger / Não aplicável]

### Autenticação e autorização
[Requisitos / Validações necessárias / Não aplicável]

### Surface de ataque
[Novos endpoints / APIs expostas / Validações de entrada / Não aplicável]

### Compliance
[LGPD / PCI / Outros / Não aplicável]

### Logs e auditoria
[O que registrar para trilha de auditoria / Não aplicável]

[Se não houver aspectos relevantes: "Não identificadas considerações de segurança críticas para esta atividade."]

## 5. Ordem de implementação

1. [Passo]
2. [Passo]

## 6. Estratégia de rollout

[Como liberar / ativar a mudança com segurança. Se o Discovery indicar feature flag, registre nome/critério de ativação; senão, descreva a estratégia de ativação escolhida.]

## 7. Estratégia de teste

[Preencher somente após a triagem. Se a resposta for Não: uma frase dizendo que testes estão fora do escopo.]

## 8. Observabilidade

### Métricas de negócio
[Indicadores de sucesso da funcionalidade]

### Métricas técnicas
[Performance, latência, taxa de erro]

### Alertas de segurança
[Tentativas de acesso não autorizado / Anomalias / Não aplicável]

### Logs
[O que será registrado / Onde acompanhar]

### Dashboards
[Onde visualizar / Não aplicável]

## 9. Pendências

- [Decisão em aberto: … / Pendência de alinhamento: … / outra lacuna objetiva]
```

Entregue este conteúdo **somente** na resposta do agente.

---

## Regras

- Seja objetivo; evite repetir texto integral de issues ou Discovery.
- Faça perguntas quando faltarem informações, em vez de inventar contexto.
- Não avance para implementação de código — esta skill encerra no plano.
- Não entregue o plano final sem confrontar decisões em aberto detectadas no Discovery/entrada; o que não for resolvido deve aparecer de forma explícita em **Pendências**.
- Não invente nem omita controles de segurança sem validar com o usuário.
- Se surgir decisão arquitetural relevante, sugira registrar com `adr-simplificado`.
