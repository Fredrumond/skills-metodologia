# Exemplos — plan-slice

## Fatia única (não fatiar)

Demanda: corrigir formatação de data no comprovante de pagamento.

Não há fronteira real (um arquivo, um comportamento). Uma fatia só.

Índice:

1. Corrigir formatação de data no comprovante

O bloco da fatia usa o template completo, com **Fora** vazio ou “nada além deste ajuste”, **Dependências: Nenhuma**, e **Pronto quando** verificável no próprio comprovante.

---

## Várias fatias (fronteira real)

Demanda: e-mail de aviso 7 dias antes da assinatura vencer, com feature flag
`subscription-expiry-email` (default off). UI fora de escopo. Testes: Sim.

Índice:

1. Persistência e flag (sem envio)
2. Job + envio (consome o contrato da Fatia 1)
3. Métricas e alerta de falha de envio

Cada fatia abaixo está num fence `markdown` (copiável). O passo de execução é o caminho equivalente, sem colar o bloco.

```markdown
### Fatia 1 — Persistência e flag

#### Contexto mínimo
Precisamos saber quem vence em 7 dias e se o aviso já foi enviado.
Nenhum e-mail sai nesta fatia.

#### Dentro
- Campo `expiry_notified_at` na assinatura
- Flag `subscription-expiry-email` cadastrada, default off
- Leitura interna: assinaturas que vencem em 7 dias e ainda não foram avisadas

#### Fora
- Job, provider de e-mail, template, métricas, UI

#### Dependências
- Nenhuma

#### Fronteira
- Pode alterar: modelo/persistência de assinatura, config da flag
- Não alterar: workers, mailer, controllers HTTP

#### Contrato de saída
- Consulta que retorna candidatos ao aviso
- Flag existente e consultável
- Campo de “já avisado” persistido, ainda não escrito pelo job

#### Ordem de implementação
1. Migração/campo `expiry_notified_at`
2. Registrar a flag default off
3. Query/repositório dos candidatos

#### Testes desta fatia
- Query: vence em 7 dias, ainda não avisado → entra
- Já avisado ou fora da janela → não entra

#### Restrições globais a respeitar
- Segurança: não logar e-mail
- Rollout: flag default off
- Observabilidade desta fatia: nenhuma

#### Pronto quando
- Migração aplicável em ambiente local
- Query coberta por teste
- Flag presente e desligada
- Nenhum e-mail enviado e nenhum worker alterado
```

```markdown
### Fatia 2 — Job e envio

#### Contexto mínimo
Com a query e a flag da Fatia 1, um job diário envia o e-mail e marca
`expiry_notified_at`. Sem a flag, o job não envia.

#### Dentro
- Job diário
- Template do e-mail (sem PII extra)
- Chamada ao mailer existente
- Escrita de `expiry_notified_at` após sucesso

#### Fora
- Mudança no modelo além de escrever o campo já criado
- Métricas/dashboard
- Endpoint HTTP, UI

#### Dependências
- Fatia 1 deve ter deixado: query de candidatos, flag, campo `expiry_notified_at`

#### Fronteira
- Pode alterar: worker/jobs, mailer (só uso), template de expiry
- Não alterar: schema/migrações, config da flag, camadas HTTP

#### Contrato de saída
- Job idempotente: não reenvia se já houver `expiry_notified_at`
- Flag off → zero envios
- Flag on → um e-mail por assinatura candidata + campo preenchido

#### Ordem de implementação
1. Template
2. Job que lê a query, checa a flag, envia, persiste
3. Idempotência no sucesso

#### Testes desta fatia
- Flag off → não chama mailer
- Flag on → chama 1x e persiste `expiry_notified_at`
- Reexecução → não reenvia

#### Restrições globais a respeitar
- Segurança: logar `subscription_id`, nunca o e-mail
- Rollout: respeitar a flag
- Observabilidade desta fatia: log de resultado por `subscription_id`

#### Pronto quando
- Testes acima passando
- Com flag off, job idle não envia
- Nenhuma rota HTTP nova
```

```markdown
### Fatia 3 — Métricas de envio

#### Contexto mínimo
Expor `expiry_emails_sent_total` e taxa de erro do provider, reusando o
painel de e-mail. Sem mudar regra de negócio.

#### Dentro
- Contador de envios e de falha
- Ponto no dashboard existente

#### Fora
- Query, flag, template, job além de instrumentar

#### Dependências
- Fatia 2 deve ter deixado: job que envia e registra sucesso/falha

#### Fronteira
- Pode alterar: job (só métricas), dashboard de e-mail
- Não alterar: persistência, flag, template, regra de candidatos

#### Contrato de saída
- Métrica incrementa só em envio efetivo (flag on + sucesso)
- Falha do provider incrementa erro, não marca `expiry_notified_at`

#### Ordem de implementação
1. Instrumentar sucesso/falha no job
2. Incluir no dashboard existente

#### Testes desta fatia
- Sucesso incrementa sent; falha incrementa error e não persiste aviso

#### Restrições globais a respeitar
- Segurança: labels da métrica sem e-mail
- Observabilidade: nomes alinhados à seção 6 do plano-pai

#### Pronto quando
- Métricas visíveis no dashboard existente
- Comportamento de negócio da Fatia 2 inalterado
```
