---
name: discovery
description: >-
  Realiza o Discovery inicial de uma demanda de software antes do planejamento
  técnico, com triagem de feature flag e considerações de segurança quando a
  informação não estiver na entrada. Use quando o usuário pedir discovery,
  levantamento inicial, escopo de demanda, refinamento de requisito, ou precisar
  estruturar uma demanda antes do planejamento técnico.
disable-model-invocation: true
---

# Skill: Discovery

## Objetivo

Transformar uma demanda em um documento de Discovery estruturado.

---

## Decisões de design

### Gerar Markdown vs. persistir em arquivo

| Ação | Quando | Onde |
|------|--------|------|
| **Entregar Markdown** | Sempre que houver contexto suficiente | Resposta do agente (padrão) |
| **Salvar em arquivo** | Status **Pronto para Planejamento? Sim**, ou pedido explícito do usuário | `docs/discovery/NNNN-titulo-curto.md` |
| **Versionar no Git** | Decisão do time no momento do commit | Fora do escopo desta skill |

**Por quê:**

- O Markdown estruturado é o núcleo da skill — força clareza de escopo, premissas e lacunas.
- Persistir todo discovery no repositório gera ruído (rascunhos, duplicação com issues, docs desatualizados).
- Arquivo em `docs/discovery/` é referência durável para demandas relevantes, alinhamento de time e rastreabilidade antes do planejamento técnico.
- Versionamento no Git não é responsabilidade da skill; o time commita quando fizer sentido.

### Contexto insuficiente ou status Bloqueado

- **Não** gere o documento completo.
- **Não** crie arquivo em `docs/discovery/`.
- Liste exatamente quais informações precisam ser obtidas antes do planejamento.

---

## Responsabilidades

- Entender a demanda.
- Resumir a regra de negócio.
- Identificar o objetivo.
- Definir escopo.
- Identificar premissas.
- Checar se a entrega será controlada por feature flag (e registrar a resposta).
- Identificar e documentar considerações de segurança.
- Levantar dúvidas.
- Identificar informações ausentes.

---

## Restrições

Nunca:

- sugerir arquitetura;
- gerar plano técnico;
- escrever código;
- criar ADR;
- estimar esforço;
- commitar ou versionar arquivos automaticamente;
- inventar, omitir ou assumir se haverá feature flag quando a informação não estiver na entrada;
- inventar ou omitir riscos de segurança quando a informação não estiver na entrada.

Caso não exista contexto suficiente, interrompa o processo e liste exatamente quais informações precisam ser obtidas antes do planejamento.

---

## Fluxo

### Etapa 1 — Compreender a demanda

Leia o contexto disponível (mensagem do usuário, issues, documentos, conversas anteriores).

Se o contexto for insuficiente para preencher as seções obrigatórias com segurança, **pare aqui**. Não gere o documento. Liste exatamente quais informações precisam ser obtidas antes do planejamento.

### Etapa 2 — Triagem de feature flag (gate)

**Antes** de estruturar o Discovery, verifique se a entrada/contexto já diz se a demanda será (ou não) controlada por feature flag.

- Se a informação **já estiver** na entrada: **não** pergunte de novo; use esse fato no preenchimento.
- Se a informação estiver **ausente**, pergunte ao usuário:

> Esta demanda será controlada por feature flag?

Não invente a resposta. Aguarde a resposta do usuário.

### Etapa 2.5 — Identificar considerações de segurança (gate)

**Antes** de estruturar o Discovery, avalie se há aspectos de segurança relevantes:

- Dados sensíveis (PII, credenciais, tokens)?
- Necessidade de autenticação/autorização?
- Exposição de APIs ou endpoints novos?
- Alteração em controles de acesso existentes?
- Requisitos de compliance (LGPD, PCI, etc.)?

Se houver algum aspecto relevante, documente na seção **Considerações de segurança**.

Se não houver aspectos de segurança significativos, registre explicitamente: `Não identificadas considerações de segurança relevantes para esta demanda.`

Não invente riscos. Se houver dúvida, pergunte ao usuário.

### Etapa 3 — Estruturar o Discovery

Com contexto suficiente (incluindo a resposta da triagem de feature flag e as considerações de segurança), preencha cada seção da saída obrigatória e **entregue o Markdown na resposta**.

Regras durante o preenchimento:

- Separe fato de suposição — premissas devem ser explícitas.
- Escopo deve deixar claro o que está **dentro** e o que está **fora**.
- Em **Premissas**, registre explicitamente: `Feature flag: Sim` ou `Feature flag: Não` (se Sim, acrescente detalhe curto se o usuário informar — nome, escopo ou critério de ativação).
- Se a resposta for **Sim** e houver impacto de escopo, reflita também em **Escopo** (Dentro/Fora) quando fizer sentido.
- Em **Considerações de segurança**, registre fatos e lacunas; não invente riscos.
- Dúvidas devem ser acionáveis (quem responde, o que precisa ser decidido).
- Informações ausentes são lacunas objetivas, não opiniões.

### Etapa 4 — Avaliar prontidão

Defina o status com base em:

- **Pronto para Planejamento** — escopo e objetivo claros; dúvidas críticas respondidas ou aceitas como risco explícito.
- **Bloqueado** — lacunas impedem planejamento seguro; liste o que falta obter.

Se o status for **Não**, entregue o Markdown na resposta com as lacunas documentadas, mas **não salve arquivo**.

### Etapa 5 — Persistir (condicional)

Salve em arquivo **somente** quando:

- o status for **Pronto para Planejamento? Sim**, ou
- o usuário pedir explicitamente para salvar.

Quando persistir, use:

```
docs/discovery/NNNN-titulo-curto.md
```

Convenções:

- `NNNN`: número sequencial com 4 dígitos (`0001`, `0002`, …).
- `titulo-curto`: slug em kebab-case, em português, descrevendo a demanda.
- Antes de criar, liste `docs/discovery/` e use o próximo número disponível.
- Exemplo: `docs/discovery/0001-notificacao-por-email.md`.

Não sobrescreva documentos existentes; se a demanda evoluir, crie um novo documento e referencie o anterior.

---

## Saída obrigatória

Gere um documento Markdown contendo exatamente estas seções:

```markdown
# Discovery: [Título da demanda]

## 1. Resumo

[Parágrafo curto com a regra de negócio e o contexto da demanda.]

## 2. Objetivo

[O que se pretende alcançar com esta entrega.]

## 3. Escopo

### Dentro

- [Item]

### Fora

- [Item]

## 4. Premissas

- Feature flag: [Sim | Não] — [detalhe curto se Sim e informado]
- [Outra premissa assumida]

## 5. Considerações de segurança

- Dados sensíveis: [Quais dados / Não aplicável]
- Autenticação/Autorização: [Requisitos / Não aplicável]
- Exposição de APIs: [Endpoints novos ou alterados / Não aplicável]
- Compliance: [LGPD, PCI, outros / Não aplicável]
- Outras considerações: [Descrição / Nenhuma identificada]

[Se não houver aspectos relevantes: "Não identificadas considerações de segurança relevantes para esta demanda."]

## 6. Dúvidas

- [Dúvida] — [por que importa / quem pode responder]

## 7. Informações ausentes

- [Lacuna objetiva que ainda precisa ser levantada]

## 8. Status

**Pronto para Planejamento?** [Sim / Não]

[Justificativa em uma ou duas frases.]
```

Por padrão, entregue este conteúdo na resposta. Salve em `docs/discovery/` apenas conforme a Etapa 5.

---

## Regras

- Seja objetivo; evite repetir texto integral de issues ou conversas.
- Não avance para planejamento técnico — este documento encerra na etapa de Discovery.
- Faça perguntas quando faltarem informações, em vez de inventar contexto.
- Não omita nem assuma feature flag: se a informação não estiver na entrada, use a Etapa 2.
- Não invente nem omita riscos de segurança: se houver dúvida, use a Etapa 2.5.
- Não force persistência em arquivo para explorações rápidas ou discoveries bloqueados.
