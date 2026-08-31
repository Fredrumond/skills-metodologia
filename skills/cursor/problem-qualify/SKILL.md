---
name: problem-qualify
description: >-
  Qualifica a qualidade de entrada de um card/demanda (0–100) antes do Discovery,
  identifica lacunas e entrevista o PM — incluindo segurança, testes e estratégia
  de entrega. Use quando o usuário pedir qualificação de card, problem-qualify,
  qualidade de entrada, refinamento com PM, ou quiser validar se uma demanda
  está pronta para Discovery.
disable-model-invocation: true
---

# Skill: Problem Qualify

## Objetivo

Garantir que a equipe comece a trabalhar a partir de uma definição clara e compartilhada do problema — medindo a qualidade da entrada, entrevistando o PM nas lacunas e liberando o card para Discovery só quando estiver pronto.

---

## Decisões de design

### Entrega apenas no chat

| Ação | Quando | Onde |
|------|--------|------|
| **Entregar Markdown** | Sempre | Resposta do agente |
| **Salvar em arquivo** | Nunca | — |
| **Versionar no Git** | Fora do escopo | — |

**Por quê:** a qualificação é um artefato de alinhamento na conversa; persistir em disco gera ruído e compete com issues/cards.

### Relação com Discovery

Esta skill é o gate **antes** de `discovery`. Não há pré-requisito formal nas outras skills; quando o status for **Pronto para Discovery? Sim**, sugira chamar `discovery` com o contexto organizado.

---

## Responsabilidades

- Entrevistar o PM.
- Identificar lacunas.
- Pedir exemplos.
- Pedir evidências.
- Identificar impacto.
- Identificar riscos de segurança.
- Avaliar necessidade de testes.
- Avaliar estratégia de entrega (feature flag, rollout).
- Organizar contexto.
- Medir qualidade da entrada (0–100).

---

## Restrições

Nunca:

- resolver o problema;
- sugerir arquitetura;
- gerar plano técnico;
- escrever código;
- criar ADR;
- estimar esforço;
- inventar evidências, impacto, critérios de sucesso, riscos de segurança, testes ou estratégia de entrega;
- persistir arquivo;
- avançar para Discovery quando o gate não for atendido.

---

## Critérios e score (0–100)

Avalie exatamente estes nove critérios. Os cinco primeiros valem **12 pontos** cada; os quatro restantes valem **10 pontos** cada, totalizando 100.

| Critério | Verde (peso cheio) | Amarelo (~50%) | Vermelho (0) | Pontos |
|----------|--------------------|----------------|--------------|--------|
| Contexto | Problema e situação claros | Parcial / vago | Ausente | 12 |
| Impacto | Quem sofre e o que muda está explícito | Genérico | Ausente | 12 |
| Evidências | Dados, exemplos ou relatos concretos | Insuficientes | Nenhuma | 12 |
| Critério de sucesso | Mensurável / verificável | Vago | Ausente | 12 |
| Escopo inicial | Dentro/fora razoável | Só “dentro” ou ambíguo | Ausente | 12 |
| Stakeholders | Papéis/pessoas identificados | Parcial | Não identificados | 10 |
| Segurança | Riscos mapeados e mitigação clara | Mencionado mas vago | Não considerado | 10 |
| Testes | Estratégia definida ou justificada | Mencionado | Não considerado | 10 |
| Estratégia de entrega | Feature flag/rollout definido | Mencionado | Não considerado | 10 |

### Detalhamento dos critérios de risco e entrega

**Segurança (10 pontos):**

- 🟢 Verde: dados sensíveis mapeados, autenticação/autorização definida, compliance considerada e/ou surface de ataque avaliada — ou justificativa clara de “não aplicável”.
- 🟡 Amarelo: segurança mencionada, mas sem detalhes de mitigação.
- 🔴 Vermelho: segurança não considerada.

**Testes (10 pontos):**

- 🟢 Verde: estratégia de teste definida (tipos, escopo) **ou** justificativa clara de por que não haverá testes.
- 🟡 Amarelo: mencionado que terá testes, mas sem estratégia.
- 🔴 Vermelho: não considerado.

**Estratégia de entrega (10 pontos):**

- 🟢 Verde: feature flag, rollout gradual ou estratégia de ativação definida.
- 🟡 Amarelo: mencionado, mas sem detalhes.
- 🔴 Vermelho: não considerado.

Pontuação por critério:

- 🟢 Verde → pontos cheios
- 🟡 Amarelo → metade dos pontos (arredondar para baixo)
- 🔴 Vermelho → 0

Score final = soma arredondada a inteiro entre 0 e 100.

### Gate — Pronto para Discovery

**Sim** somente quando:

1. score ≥ **80**, **e**
2. **nenhum** critério vermelho.

Caso contrário: **Não**.

---

## Fluxo

### Etapa 1 — Ler o card

Leia o contexto disponível (mensagem do usuário, card, issue, bug, incidente, feature, conversas anteriores).

### Etapa 2 — Avaliar critérios

Classifique cada critério (🟢 / 🟡 / 🔴), calcule o score e determine o status do gate.

### Etapa 3a — Se NÃO pronto

- Liste as lacunas de forma objetiva.
- Gere perguntas ao PM (exemplos, evidências, impacto, sucesso, escopo, stakeholders, segurança, testes, estratégia de entrega).
- **Não** sugira avançar para Discovery.
- Em rodadas seguintes da mesma conversa, incorpore as respostas e **reavalie** score e critérios.

### Etapa 3b — Se pronto

- Entregue o resumo e o contexto organizado para Discovery.
- Declare **Pronto para Discovery? Sim**.
- Sugira chamar a skill `discovery`.

---

## Saída obrigatória

Gere um documento Markdown contendo exatamente estas seções:

```markdown
# Qualificação: [Título]

## 1. Score de entrada

**XX/100** — Pronto para Discovery? [Sim / Não]

- 🟢/🟡/🔴 Contexto — [nota breve]
- 🟢/🟡/🔴 Impacto — [nota breve]
- 🟢/🟡/🔴 Evidências — [nota breve]
- 🟢/🟡/🔴 Critério de sucesso — [nota breve]
- 🟢/🟡/🔴 Escopo inicial — [nota breve]
- 🟢/🟡/🔴 Stakeholders — [nota breve]
- 🟢/🟡/🔴 Segurança — [nota breve]
- 🟢/🟡/🔴 Testes — [nota breve]
- 🟢/🟡/🔴 Estratégia de entrega — [nota breve]

## 2. Resumo do problema

[Parágrafo curto com o que se entende até agora, sem inventar fatos.]

## 3. Lacunas

- [Lacuna objetiva]

## 4. Perguntas ao PM

[Inclua esta seção somente se Pronto para Discovery? Não.]

- [Pergunta objetiva ao PM]

## 5. Contexto organizado para Discovery

[Inclua esta seção somente se Pronto para Discovery? Sim.]

[Contexto compartilhado pronto para alimentar a skill discovery — inclua o que já foi alinhado sobre segurança, testes e estratégia de entrega.]

Próximo passo sugerido: chamar a skill `discovery`.
```

Entregue este conteúdo **somente** na resposta do agente.

---

## Regras

- Seja objetivo; evite repetir texto integral do card.
- Faça perguntas quando faltarem informações, em vez de inventar contexto.
- Uma pergunta por lacuna crítica; agrupe o mínimo necessário para destravar o gate.
- Não force avanço para Discovery com score < 80 ou com critério vermelho.
- Esta skill encerra na qualificação — Discovery, plano e ADR ficam fora do escopo.
