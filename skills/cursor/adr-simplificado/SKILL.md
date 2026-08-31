---
name: adr-simplificado
description: >-
  Registra decisões de arquitetura em formato ADR simplificado, com triagem
  prévia e estrutura objetiva. Use quando o usuário pedir um ADR, registro de
  decisão de arquitetura, ou precisar documentar trade-offs, padrões,
  integrações ou escolhas tecnológicas de longo prazo.
disable-model-invocation: true
---

# Skill: ADR Simplificado

## Objetivo

Registrar decisões de arquitetura de forma objetiva e consistente.

---

## Local e nome do arquivo

Salve cada ADR em:

```
docs/adr/NNNN-titulo-curto.md
```

Convenções:

- `NNNN`: número sequencial com 4 dígitos (`0001`, `0002`, …).
- `titulo-curto`: slug em kebab-case, em português, descrevendo a decisão.
- Antes de criar, liste `docs/adr/` e use o próximo número disponível.
- Exemplo: `docs/adr/0001-usar-fila-para-campanhas.md`.

Não sobrescreva ADRs existentes; se a decisão mudar, crie um novo ADR e referencie o anterior.

---

## Etapa 1 - Triagem

Antes de criar um ADR, responda às perguntas abaixo.

### A decisão altera a arquitetura do sistema?
Exemplos:
- Novo padrão arquitetural
- Nova integração relevante
- Nova tecnologia
- Mudança na comunicação entre componentes

### A decisão terá impacto em futuras implementações?

Se esta decisão criar um padrão que outras funcionalidades deverão seguir, considere criar um ADR.

### Existem trade-offs importantes?

Exemplos:
- Performance x simplicidade
- Custo x escalabilidade
- Acoplamento x velocidade de entrega
- Consistência x disponibilidade

### A decisão é difícil de reverter?

Quanto maior o custo de mudança futura, maior a necessidade de um ADR.

### Existe uma alternativa relevante que foi descartada?

Se sim, registre o motivo da escolha.

### Esta decisão provavelmente será questionada daqui a 6 ou 12 meses?

Se a resposta for sim, um ADR pode evitar rediscussões.

---

## Não criar ADR quando

Não gere um ADR para:

- Correções de bugs
- Refatorações internas
- Mudanças pequenas de código
- Alterações de regra de negócio sem impacto arquitetural
- Troca de implementação sem alterar a arquitetura
- Ajustes de configuração

Nestes casos, sugira registrar a informação na Issue, Pull Request ou documentação funcional.

---

## Criar ADR quando

Crie um ADR quando houver:

- Decisão arquitetural
- Adoção de novo padrão
- Escolha entre tecnologias
- Definição de integrações
- Mudança estrutural no sistema
- Decisão com impacto de longo prazo
- Definição de um padrão que deverá ser seguido pelo restante do projeto

---

## Estrutura do ADR

Sempre inclua a data de geração no topo do documento, no formato `DD/MM/AAAA`:

```markdown
**Data:** 30/03/2026
```

Use a data do dia em que o ADR for criado.

### Contexto

Qual problema estamos resolvendo?

### Opções consideradas

Liste as alternativas avaliadas.

### Decisão

Qual foi a decisão tomada?

### Justificativa

Por que essa opção foi escolhida?

### Consequências

#### Benefícios

#### Riscos

#### Débitos técnicos

#### Próximos passos (opcional)

---

## Regras

- Máximo de uma página.
- Seja objetivo.
- Sempre registre a data de geração (`**Data:** DD/MM/AAAA`).
- Evite repetir informações da Issue.
- Faça perguntas caso faltem informações.
- Caso conclua que um ADR não é necessário, explique o motivo e sugira onde registrar a decisão.
