---
name: delivery-summary
description: >-
  Gera o corpo do PR (Summary, Contexto/Context, Test plan, Related) fácil de
  copiar e um resumo de negócio estruturado para Slack, e-mail ou card —
  incluindo segurança e observabilidade quando relevantes — com triagem de
  idioma e de links (card/documentos). Use quando o usuário pedir
  delivery-summary, descrição de PR, handoff de entrega, resumo para negócio,
  ou comunicação pós-implementação da mudança.
disable-model-invocation: true
---

# Skill: Delivery Summary

## Objetivo

Após a implementação, entregar no chat o texto do PR (fácil de copiar e colar) e um resumo de negócio colável — no mesmo idioma escolhido pelo usuário.

---

## Decisões de design

### Entrega apenas no chat

| Ação | Quando | Onde |
|------|--------|------|
| **Entregar Markdown** | Sempre | Resposta do agente |
| **Salvar em arquivo** | Nunca | — |
| **Versionar no Git** | Fora do escopo | — |
| **Criar PR no GitHub** | Nunca | — |

**Por quê:** o usuário cola o texto no PR, Slack, e-mail ou card; a skill não compete com `gh` nem com templates do repositório.

### Corpo do PR colável

O **Bloco A** sai em um único fence Markdown (` ```markdown ` … ` ``` `), **sem prosa dentro do fence**, para um Ctrl+C limpo. O Bloco B fica fora desse fence.

### Idioma único

A triagem de idioma (`português` | `inglês`) vale para **os dois blocos** (corpo do PR e resumo de negócio). Texto e rótulos no mesmo idioma.

### Relação com outras skills

Etapa 3 do fluxo (pós-implementação). Se a mudança afetar comando, fluxo ou serviço relevante para onboarding, **sugira** chamar `document-technical-reference` — não execute.

---

## Responsabilidades

- Triar idioma e referências (card / documentos) antes de gerar.
- Montar o corpo do PR a partir do diff/branch/commits, card e Discovery quando existirem.
- Entregar o corpo do PR em formato fácil de copiar e colar.
- Montar o resumo de negócio sem jargão técnico, incluindo aspectos de segurança e observabilidade quando relevantes.
- Manter Summary simples e objetivo (poucos bullets; agrupar por tema).

---

## Restrições

Nunca:

- abrir ou criar PR no GitHub (`gh pr create` ou equivalente);
- fazer code review;
- escrever referência técnica (skill `document-technical-reference`);
- inventar card, Discovery, ADR, links ou resultados de teste;
- inventar proteções de segurança ou instrumentação que não estejam no diff/contexto;
- gerar os blocos sem idioma definido;
- gerar os blocos sem concluir a triagem de referências (exceto quando o usuário já informou ou disse que não há).

Caso o contexto da mudança seja insuficiente, **pare**, faça perguntas e liste o que falta.

---

## Fluxo

### Etapa 1 — Triagem (gate)

**Antes** de gerar qualquer bloco, faça a triagem abaixo (pule só o que o usuário **já** tiver informado na mensagem).

1. **Idioma**

> Em qual idioma gerar o PR e o resumo de negócio? Português ou inglês?

2. **Referências**

> Envie o link do card e de qualquer outro documento que deseje referenciar/anexar no PR (ou diga que não há).

Não invente as respostas. Aguarde.

A escolha de idioma vale para **corpo do PR e resumo de negócio**. Os links informados alimentam **Contexto** e **Related**.

### Etapa 2 — Coletar contexto

Use o que estiver disponível:

- diff, branch e commits da conversa ou do working tree;
- card (link ou id) — da triagem ou da mensagem;
- outros documentos/links da triagem;
- Discovery em `docs/discovery/` ou na conversa;
- Plan na conversa, se houver;
- ADR citado, se relevante.

Não invente links. Se card ou Discovery não existirem, registre "não informado" / "not provided" (ou omita Discovery quando não houver).

### Etapa 3 — Entregar os dois blocos

Entregue **somente na resposta**, nesta ordem:

1. **Bloco A — Corpo do PR** (idioma escolhido): um único fence `markdown`, sem prosa dentro do fence.
2. **Bloco B — Resumo de negócio** (mesmo idioma): fora do fence do Bloco A.

Pode haver uma linha curta de orientação **fora** dos fences (ex.: "Cole o bloco abaixo no PR"), mas o conteúdo colável do PR fica só dentro do fence do Bloco A.

Ao construir o Bloco B:

- Identifique no diff/commits aspectos de segurança implementados (autenticação, validações, criptografia, etc.).
- Identifique instrumentação adicionada (logs, métricas, traces, alertas).
- Consulte o Discovery e o Plan (se disponíveis) para contexto de segurança e observabilidade.
- Omita seções vazias; foque apenas no que foi efetivamente entregue.

Se a mudança afetar comando/fluxo/serviço relevante para onboarding, ao final sugira chamar `document-technical-reference`.

---

## Saída obrigatória

### Bloco A — Corpo do PR

Entregue o Bloco A **dentro** de um fence assim (conteúdo interno = só o Markdown do PR):

````markdown
```markdown
## Summary
…
```
````

**Cabeçalhos em português:** `Summary` / `Contexto` / `Test plan` / `Related`  
**Cabeçalhos em inglês:** `Summary` / `Context` / `Test plan` / `Related`

Conteúdo interno do fence:

```markdown
## Summary

- [Bullets simples e objetivos: o quê mudou e por quê / impacto]
- [Sem limite rígido; preferir poucos — só o necessário para o revisor entender]
- [Se houver muitos pontos, agrupar por tema; não um bullet por arquivo]

## Contexto

- **Card:** [link ou id; ou "não informado" / "not provided"]
- **Discovery:** [path ou link se existir; omitir a linha se não houver]
- **Documentos:** [links informados na triagem; omitir a linha se não houver]

## Test plan

- [ ] [Item acionável: comando, ambiente, o que observar]
- [ ] [Item]

## Related

<!-- Links do card/docs/issues. Use Closes #N quando couber. Omitir a seção se vazia. -->
```

Em inglês, use `## Context` no lugar de `## Contexto`. Marque `[x]` só no que o usuário já confirmou ter testado.

### Bloco B — Resumo de negócio

Bloco separado, colável (Slack, e-mail, card), **no mesmo idioma** do PR.

Título sugerido: `## Resumo de negócio` ou `## Business summary`.

Estrutura (incluir somente seções relevantes):

```markdown
## Resumo de negócio

### O que foi entregue
[2–3 frases em linguagem de negócio sobre a mudança]

### Por que importa
[Impacto no negócio, usuários, eficiência operacional]

### O que muda na prática
[Novos comportamentos, fluxos alterados, experiência do usuário]

### Segurança
[Incluir somente se houver aspectos relevantes:]
- Proteções implementadas
- Dados protegidos
- Controles de acesso
- Compliance atendida

### Observabilidade
[Sempre incluir quando houver instrumentação:]
- Métricas disponíveis: [ex.: taxa de sucesso, latência]
- Logs: [o que foi instrumentado]
- Alertas: [condições monitoradas]
- Onde acompanhar: [dashboard, ferramenta, link]

### Próximos passos
[Incluir somente se houver:]
- Fase 2 planejada
- Rollout gradual
- Ativação programada

### Atenção
[Incluir somente se houver:]
- Riscos conhecidos
- Limitações
- Dependências
- Período de observação
```

Em inglês, use os rótulos: `## Business summary`, `### What was delivered`, `### Why it matters`, `### What changes in practice`, `### Security`, `### Observability`, `### Next steps`, `### Watch-outs`.

**Diretrizes:**

- Omita seções vazias ou não aplicáveis.
- Use linguagem de negócio; evite nomes de classes ou detalhes de implementação.
- Priorize clareza sobre completude.
- Seções **O que foi entregue**, **Por que importa** e **O que muda na prática** são sempre obrigatórias.
- **Observabilidade** é obrigatória quando houver instrumentação.
- **Segurança** é obrigatória quando houver aspectos relevantes de proteção ou compliance.

---

## Regras

- Seja objetivo; não despeje o diff no Summary.
- Faça perguntas quando faltarem informações, em vez de inventar contexto.
- Conclua a triagem de idioma e de referências antes de gerar os blocos.
- O corpo do PR (Bloco A) deve ser fácil de copiar e colar: um fence, sem prosa interna.
- No Bloco B, não invente segurança ou observabilidade ausentes no contexto da entrega.
- Esta skill encerra nos dois blocos no chat — não abre PR nem grava arquivo.
