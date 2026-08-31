---
name: document-technical-reference
description: >-
  Cria ou revisa documentação de referência técnica no formato de tópicos
  claros para onboarding. Use quando o usuário pedir para documentar,
  atualizar ou revisar uma referência a partir de um caminho de arquivo,
  pasta, comando, rotina, serviço ou fluxo do projeto. Aplica a estrutura
  de tópico (para que serve, como funciona, como rodar) ao que for passado,
  sem limitar-se a scheduled jobs.
disable-model-invocation: true
---

# Documentar referência técnica

Padroniza documentação objetiva para onboarding: cada assunto vira **um tópico**
fácil de escanear. O agente documenta **somente o que o usuário indicar**
(caminho de arquivo, pasta, comando, classe, fluxo, etc.), no formato abaixo.

Não inventar escopo. Se o usuário passar um Command, documenta aquele Command.
Se passar uma pasta, documenta o que for pedido dentro dela. Se pedir revisão,
revisa o trecho indicado.

## Quando usar

- Documentar uma referência técnica a partir de um caminho ou símbolo
- Atualizar um tópico existente depois de mudança de código
- Revisar se uma doc de referência está clara e fiel ao código

## Entrada esperada

O usuário deve informar (ou o agente deve pedir se faltar):

1. **O quê documentar** — path, comando, classe, fluxo
2. **Onde gravar** (opcional) — arquivo em `docs/` (ex.: `docs/references/….md`)
3. **Modo** — criar, atualizar ou revisar

Se o destino não for informado, sugerir um path em `docs/references/` e confirmar antes de escrever.

## Modos

### Criar / atualizar

1. Ler o caminho/código indicado (e só as dependências necessárias para entender o fluxo).
2. Extrair fatos do código: responsabilidade, passos reais, dependências, como executar, monitors/logs se existirem.
3. Procurar ADRs/docs relacionados em `docs/` e linkar só se existirem e forem relevantes.
4. Escrever ou atualizar **um tópico por assunto**, no template abaixo.
5. Se o arquivo de destino tiver vários tópicos, manter um **índice** no topo (agrupado por domínio quando fizer sentido).
6. Se fizer sentido para onboarding, garantir link no `README.md` apontando **direto** para o arquivo da referência (sem README intermediário vazio em `docs/references/`).

### Revisar

Comparar o tópico com o código indicado e aplicar o checklist. Reportar gaps objetivos. Só reescrever se o usuário pedir correção.

#### Checklist de revisão

- [ ] Escopo limitado ao que o usuário indicou (sem inventar assuntos)
- [ ] Título descreve o propósito em português (não o nome técnico cru)
- [ ] Tabela completa: Comando / entrada, Quando, Monitor, Log, Código
- [ ] Campos Monitor / Log / Código refletem o que existe no código (ou N/A / Não)
- [ ] **Para que serve**: 1–3 frases em linguagem humana, alinhadas à responsabilidade real
- [ ] **Como funciona**: 4–7 passos reais do fluxo, sem dump de código
- [ ] **Rodar na mão**: comando local real e aplicável (ou explicação clara se não houver)
- [ ] Fatos (passos, dependências, gatilhos) batem com o código atual
- [ ] Links para ADRs/docs só se existirem e forem relevantes
- [ ] Tópico fácil de escanear para onboarding
- [ ] Se o arquivo tem vários tópicos: índice no topo presente e útil
- [ ] Se faz sentido para onboarding: `README.md` linka direto para o arquivo da referência

## Template obrigatório (cada assunto = um tópico)

Título = propósito em português (não o nome técnico cru do símbolo).

```markdown
## N. Título em português

| | |
|---|---|
| **Comando / entrada** | signature, rota, classe ou ponto de entrada |
| **Quando** | frequência, gatilho ou “sob demanda” |
| **Monitor** | Cronitor/outro ou Não / N/A |
| **Log** | path de log ou N/A |
| **Código** | path principal do código |

**Para que serve**

1–3 frases: o problema que resolve, em linguagem humana.

**Como funciona**

1. Passo real do fluxo
2. …
3. … (4–7 passos; sem dump de código)

**Rodar na mão**

\`\`\`bash
# comando real de execução local, se aplicável
\`\`\`
```
