---
name: plan-slice
description: >-
  Gera um plano técnico em Markdown no chat, fatiado em unidades de execução
  isoladas para o próximo agente (escopo, fronteira, contrato de saída, testes
  da fatia e critério de pronto), com restrições globais de segurança, rollout e
  observabilidade. Use quando o usuário pedir plan-slice, plano fatiado, fatias
  de execução, escopo isolado para o agente, ou quiser transformar uma
  demanda/discovery em plano executável uma fatia por vez.
disable-model-invocation: true
---

# Skill: Plan Slice

## Objetivo

Transformar uma demanda (e, quando disponível, um Discovery) em um plano técnico
cuja unidade de trabalho é a **fatia**: um brief auto-contido, pequeno o
suficiente para um agente implementar isolado, sem a demanda inteira.

Não substitui a skill `plan`. Use `plan` para um plano único da atividade; use
esta skill para fatiar a execução.

Exemplo preenchido: [examples.md](examples.md).

---

## Decisões de design

### Entrega apenas no chat

| Ação | Quando | Onde |
|------|--------|------|
| **Entregar Markdown** | Sempre | Resposta do agente |
| **Salvar em arquivo** | Nunca | — |
| **Versionar no Git** | Fora do escopo | — |

**Por quê:** o plano-pai é alinhamento; cada fatia é um bloco Markdown copiável
**ou** o insumo do passo de execução. Persistir gera ruído e compete com issues/PRs.

### Fatia = unidade de execução

- Fatia é o que o próximo agente recebe — não um ticket, PR ou estimativa.
- Só fatiar quando houver **fronteira real** (contrato, módulo, flag, camada).
- Demanda pequena → **Fatia única**, com o mesmo template.
- Preferir 2–5 fatias. Se parecer precisar de mais, questione se a fronteira é real.
- A ordem de implementação vive **dentro** da fatia, não no plano-pai.

### Discovery (opcional)

Não há pré-requisito formal com `discovery`. Se houver Discovery na conversa ou
em `docs/discovery/`, use-o como contexto. Caso contrário, planeje com o que o
usuário passar.

### Modo Plan do Cursor

Entregue o Markdown na resposta; **não** crie arquivo. O artefato interno do
Cursor (`CreatePlan`) é opcional e **separado** — não misture os dois.

---

## Responsabilidades

- Compreender a demanda e alinhar o caminho ao `QUICK_START_GUIDE` (se existir).
- Confrontar decisões em aberto antes do plano final.
- Definir restrições globais (segurança, rollout, observabilidade).
- Triar testes com o usuário **antes** de preencher testes das fatias.
- Quebrar em fatias isoláveis, cada uma auto-contida para o agente executor.
- Registrar pendências explícitas.

---

## Restrições

Nunca:

- escrever código de produção;
- criar ADR (sugerir `adr-simplificado`);
- persistir o plano em disco;
- inventar padrão do projeto se o `QUICK_START_GUIDE` não existir;
- inventar a resposta da triagem de testes;
- inventar ou omitir controles de segurança sem validar com o usuário;
- entregar o plano final fechando decisões em aberto sem questionar;
- criar fatias sem fronteira real;
- pedir ao agente executor que leia o plano inteiro para trabalhar uma fatia;
- criar ticket, estimativa, branch ou PR.

Caso o contexto seja insuficiente, **pare**, faça perguntas e liste o que falta.

---

## Fluxo

### Etapa 1 — Compreender o contexto

Leia a mensagem do usuário, Discovery (se houver), issues e documentos relevantes.

### Etapa 2 — Ler o padrão do projeto

Procure em `docs/` um arquivo `QUICK_START_GUIDE` (aceite `.md` e variações de case).

- Se existir: use no **Caminho escolhido**.
- Se não existir: continue; registre em **Pendências** se a ausência for lacuna.

### Etapa 3 — Decisões em aberto (gate)

**Antes** do plano final, detecte decisões em aberto. Fontes típicas:

- **Dúvidas** e **Informações ausentes** do Discovery;
- **Considerações de segurança** do Discovery (lacunas);
- status **Pronto para Planejamento? Não**;
- escolhas ainda não fechadas na mensagem do usuário.

Se não houver, siga para a Etapa 4.

Se houver:

1. **Pare** — não entregue o plano final.
2. Liste cada decisão (o que decidir / por que importa).
3. **Pergunte** ao usuário.
4. O que for resolvido entra no plano.
5. O que permanecer aberto vai para **Pendências** (`Decisão em aberto: …`).

### Etapa 4 — Triagem de testes (gate)

**Antes** de preencher testes das fatias, pergunte:

> Esta atividade / este projeto deve conter testes?

Não invente a resposta. Aguarde.

- **Sim:** cada fatia declara os testes **dela**.
- **Não:** em cada fatia, uma frase: testes fora do escopo desta atividade.

### Etapa 5 — Decidir fatias

Fatie somente com fronteira real. Exemplos válidos: schema → API → UI; flag off
→ comportamento atrás da flag; contrato estável entre backend e frontend.

Não fatie: um bug num arquivo, um endpoint + teste, um ajuste pontual → **Fatia única**.

Para cada fatia, preencha o template da saída. O bloco deve ser executável
sozinho: o agente da Fatia N não precisa do plano-pai nem das outras fatias,
exceto o **contrato de saída** da anterior, repetido em **Dependências**.

### Etapa 6 — Entregar

Preencha a saída obrigatória e entregue **somente na resposta**.

- Não crie arquivo. Não commit.
- Entregue cada fatia num fence `markdown` separado, para copiar o bloco.
- Inclua **Como executar** como caminho equivalente (sem copiar o Markdown).
- Repita nas fatias só as restrições globais que o executor precisa respeitar.
- Não antecipe trabalho de fatias seguintes dentro de uma fatia anterior.

---

## Saída obrigatória

```markdown
# Plano (fatias): [Título da atividade]

## 1. Objetivo

[O que se pretende alcançar com esta entrega.]

## 2. Estratégia

[Abordagem geral.]

## 3. Caminho escolhido

[Caminho técnico e por quê.]

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
[O que registrar / Não aplicável]

[Se não houver aspectos relevantes: "Não identificadas considerações de segurança críticas para esta atividade."]

## 5. Estratégia de rollout

[Como liberar / ativar. Se o Discovery indicar feature flag, registre nome/critério.]

## 6. Observabilidade

### Métricas de negócio
[Indicadores de sucesso]

### Métricas técnicas
[Performance, latência, taxa de erro]

### Alertas de segurança
[Tentativas de acesso não autorizado / Anomalias / Não aplicável]

### Logs
[O que registrar / Onde acompanhar]

### Dashboards
[Onde visualizar / Não aplicável]

## 7. Fatias de execução

[Índice: uma linha por fatia — nome + dependência.]

[Em seguida, um bloco por fatia, cada um dentro de um fence `markdown` para copiar.]
```

Template de **cada** fatia (incluindo Fatia única). Entregue cada bloco num fence
`markdown` separado:

```markdown
### Fatia N — [nome]

#### Contexto mínimo
[2–4 frases. Sem repetir o plano inteiro.]

#### Dentro
- [o que esta fatia entrega]

#### Fora
- [o que é proibido nesta fatia]

#### Dependências
- Nenhuma | Fatia N-1 deve ter deixado: [contrato de saída da anterior]

#### Fronteira
- Pode alterar: [paths / módulos]
- Não alterar: [paths / módulos]

#### Contrato de saída
- [API, tabela, flag, comportamento observável que a próxima fatia pode assumir]

#### Ordem de implementação
1. [passo desta fatia]
2. [passo desta fatia]

#### Testes desta fatia
[Tipos e casos desta fatia] | Fora de escopo (triagem = Não)

#### Restrições globais a respeitar
- Segurança: […]
- Rollout/flag: […]
- Observabilidade desta fatia: […] | Nenhuma nesta fatia

#### Pronto quando
- [critério verificável sem implementar as outras fatias]
```

Feche o documento com:

```markdown
## 8. Pendências

- [Decisão em aberto: … / Pendência de alinhamento: … / nenhuma]

## Como executar

Dois caminhos equivalentes:

1. **Copiar o Markdown** — nova sessão: cole **somente** o bloco da próxima fatia pendente.
2. **Passo de execução** — nesta ou em outra sessão: peça para executar **somente** a próxima fatia pendente.

Em ambos:

- Só avance quando o **Pronto quando** da fatia atual for verdadeiro.
- Não antecipar o trabalho das fatias seguintes.
```

---

## Regras

- Seja objetivo; evite repetir texto integral de issues ou Discovery.
- Faça perguntas quando faltarem informações, em vez de inventar contexto.
- Não avance para implementação — esta skill encerra no plano fatiado.
- Cada fatia deve ser executável isolada; se o agente ainda precisar do plano-pai,
  a fatia está incompleta.
- Não invente nem omita controles de segurança sem validar com o usuário.
- Se surgir decisão arquitetural relevante, sugira `adr-simplificado`.
