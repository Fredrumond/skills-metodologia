---
name: code-review
description: >-
  Revisa código implementado contra checklist dual-layer (universal + contextual), 
  cruzando com Discovery/Plan quando existirem, classificando achados por severidade 
  (BLOQUEADOR/AVISO/SUGESTÃO), e emitindo veredicto estruturado antes do delivery-summary. 
  Use quando o usuário pedir code review, revisão de código, revisão da implementação, 
  ou estiver pronto para validar mudanças antes de criar o PR.
disable-model-invocation: true
---

# Skill: Code Review

## Objetivo

Revisar a implementação realizada antes do `delivery-summary`, funcionando como gate automatizado no pipeline de entrega.

---

## Decisões de design

### Entrega apenas no chat

| Ação | Quando | Onde |
|------|--------|------|
| **Entregar Markdown** | Sempre | Resposta do agente |
| **Salvar em arquivo** | Nunca | — |
| **Reescrever código** | Nunca | — |

**Por quê:** a skill valida e orienta; não implementa nem persiste. O parecer estruturado guia ajustes ou libera para o `delivery-summary`.

### Posição no pipeline

Etapa 2.5 do fluxo (entre implementação e delivery-summary):

```
Discovery → Plan → Implementação → code-review → delivery-summary → PR
```

Se houver bloqueadores ou ajustes necessários, retorna para Implementação.

### Relação com outras skills

- Consome contexto de `discovery` (se existir em `docs/discovery/`)
- Consome contexto de `plan` (se existir na conversa)
- Precede `delivery-summary` (não gera corpo de PR)

---

## Responsabilidades

- Rodar `git diff HEAD` e `git log --oneline -10` automaticamente.
- Buscar Discovery mais recente em `docs/discovery/`.
- Buscar Plan na conversa (se houver).
- Aplicar checklist dual-layer (universal + contextual).
- Detectar anti-padrões ativamente.
- Classificar achados por severidade (BLOQUEADOR, AVISO, SUGESTÃO).
- Emitir veredicto: **Aprovado** / **Aprovado com ressalvas** / **Bloqueado**.
- Sugerir próximo passo.

---

## Restrições

Nunca:

- reescrever ou alterar código;
- commitar ou abrir PR;
- gerar corpo de PR (responsabilidade do `delivery-summary`);
- inventar contexto de segurança, testes ou observabilidade que não estejam no Discovery/Plan/diff;
- inventar o idioma de saída (seguir a lógica de triagem definida).

Caso o diff esteja vazio (nada a revisar), informe e encerre.

---

## Fluxo

### Etapa 1 — Coletar contexto

Execute em paralelo:

1. `git diff HEAD` — mudanças não commitadas (working tree + staged)
2. `git log --oneline -10` — histórico recente de commits
3. Listar `docs/discovery/` — buscar o Discovery mais recente (se existir)
4. Verificar se há Plan na conversa

Se o diff estiver vazio, informe e encerre: "Nenhuma mudança detectada para revisar."

### Etapa 2 — Triagem de idioma (gate)

Lógica:

1. **Existe Discovery em `docs/discovery/`?**
   - Sim → detecte o idioma do documento (português ou inglês) → use esse idioma para toda a saída.
   - Não → pergunte ao usuário:

> Em qual idioma gerar o code review? Português ou inglês?

Não invente a resposta. Aguarde.

O idioma escolhido vale para **todo o parecer** (títulos, seções, achados).

### Etapa 3 — Revisar (checklist dual-layer)

#### Camada 1 — Universal (sempre aplicar)

Revise o diff contra estes critérios:

**Lógica e corretude**
- A implementação faz o que o Plan/Discovery descreve?
- Casos de borda tratados (null, lista vazia, concorrência, timeout)?
- Nenhuma regressão óbvia nos fluxos adjacentes?

**Segurança**
- Segredos/credenciais hardcoded? (tokens, senhas, chaves de API)
- Validação de input em fronteiras (API, formulário, evento externo)?
- Controles de acesso condizentes com o Discovery?
- Dados sensíveis em logs (PII, CPF, email, tokens)?

**Qualidade de código**
- Funções com responsabilidade única (SRP)?
- Duplicação desnecessária (DRY)?
- Nomes descritivos e sem abreviações crípticas?
- Comentários explicam "por quê", não "o quê"?

**Tratamento de erros**
- Erros capturados e propagados corretamente?
- Mensagens de erro informativas sem vazar stack/dados internos?
- Sem `catch` vazio ou que apenas faz `console.log`?

**Testes**
- Se a estratégia de teste estava no Plan: novos caminhos cobertos?
- Mocks não substituem regra de negócio real?
- Testes assertam comportamento, não apenas chamadas de stub?

**Observabilidade**
- Logs e métricas planejados no Plan foram instrumentados?
- Logs não expõem PII?
- Métricas de negócio e técnicas estão presentes?

**Rollout**
- Feature flag implementada se o Discovery definiu como premissa?
- Rollback possível sem migração adicional?

#### Camada 2 — Contextual (quando houver Discovery/Plan)

Quando Discovery ou Plan existirem, cruze automaticamente:

- **Seção "Considerações de segurança" do Discovery** → gere checklist específico.
- **Seção "Estratégia de teste" do Plan** → valide se foi cumprida.
- **Seção "Observabilidade" do Plan** → valide instrumentação.
- **Dúvidas em aberto no Discovery** → foram decididas e implementadas?

#### Anti-padrões (detecção ativa)

Consulte [anti-patterns.md](anti-patterns.md) para catálogo completo. Sinalize ativamente:

- Segredo hardcoded
- Silent fail (catch vazio ou só console.log)
- God function (função com múltiplas responsabilidades)
- Teste que testa o mock
- Feature flag ausente (quando Discovery marcou Sim)
- Log de dados sensíveis
- TODO/FIXME no caminho crítico
- Dependência não declarada (import sem estar no manifesto)
- PR gigante sem contexto (diff massivo sem Discovery/Plan referenciado)

### Etapa 4 — Classificar achados

Para cada achado, atribua severidade:

- **BLOQUEADOR** — impede merge; precisa ser resolvido antes do `delivery-summary`.
- **AVISO** — deve ser endereçado; pode ser aceito como risco explícito após discussão.
- **SUGESTÃO** — melhoria opcional; não bloqueia.

### Etapa 5 — Emitir veredicto

Com base nos achados:

- **Aprovado** — nenhum bloqueador; pode seguir para `delivery-summary`.
- **Aprovado com ressalvas** — avisos presentes; recomenda endereçar, mas não bloqueia.
- **Bloqueado** — pelo menos um bloqueador; lista o que corrigir antes de prosseguir.

### Etapa 6 — Entregar o parecer

Gere a saída obrigatória e entregue **somente na resposta**.

---

## Saída obrigatória

Gere um documento Markdown contendo exatamente estas seções (títulos variam conforme idioma):

### Em português

```markdown
# Code Review: [título da mudança]

## Veredicto: [Aprovado | Aprovado com ressalvas | Bloqueado]

## Achados

### BLOQUEADOR
- [ ] [Descrição] — [arquivo:linha se possível] — [por que bloqueia]

### AVISO
- [ ] [Descrição] — [arquivo:linha se possível] — [impacto]

### SUGESTÃO
- [ ] [Descrição] — [arquivo:linha se possível]

[Omitir seções vazias.]

## Cruzamento com Discovery/Plan

- **Discovery:** [caminho do arquivo | não encontrado]
- **Dúvidas em aberto endereçadas:** [Sim | Não | Parcialmente | N/A]
- **Segurança do Discovery:** [coberta | lacuna identificada | N/A]
- **Observabilidade do Plan:** [instrumentada | ausente | N/A]

## Próximo passo

[Se Aprovado:]
Implementação validada. Chame `/delivery-summary` para gerar o corpo do PR.

[Se Aprovado com ressalvas:]
Revisar os avisos listados. Se aceitar os riscos, pode seguir para `/delivery-summary`.

[Se Bloqueado:]
Corrija os bloqueadores listados e execute `/code-review` novamente.
```

### Em inglês

```markdown
# Code Review: [change title]

## Verdict: [Approved | Approved with reservations | Blocked]

## Findings

### BLOCKER
- [ ] [Description] — [file:line if possible] — [why it blocks]

### WARNING
- [ ] [Description] — [file:line if possible] — [impact]

### SUGGESTION
- [ ] [Description] — [file:line if possible]

[Omit empty sections.]

## Discovery/Plan Cross-check

- **Discovery:** [file path | not found]
- **Open questions addressed:** [Yes | No | Partially | N/A]
- **Discovery security:** [covered | gap identified | N/A]
- **Plan observability:** [instrumented | missing | N/A]

## Next step

[If Approved:]
Implementation validated. Call `/delivery-summary` to generate the PR body.

[If Approved with reservations:]
Review the warnings listed. If you accept the risks, proceed to `/delivery-summary`.

[If Blocked:]
Fix the blockers listed and run `/code-review` again.
```

---

## Regras

- Seja objetivo; não reproduza o diff inteiro no parecer.
- Aponte para `arquivo:linha` sempre que possível.
- Não invente contexto — se não há Discovery/Plan, marque como "não encontrado" / "not found".
- Omita seções vazias (se não houver bloqueadores, não inclua `### BLOQUEADOR`).
- A triagem de idioma é obrigatória; não gere saída antes de definir o idioma.
- Esta skill encerra no parecer — não implementa correções nem abre PR.
- Se houver bloqueadores, o fluxo retorna para implementação; não sugira `delivery-summary`.
