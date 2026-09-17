# Engenharia Aumentada

> **Status:** artigo em evolução (rascunho alinhado à **v1.0** das skills).  
> **Escopo deste rascunho:** escrito **antes** de `plan-slice` e `code-review` (**v1.1**). O [README](../../README.md) e o [Changelog](../../README.md#changelog) refletem o pacote atual; este texto será acrescido conforme a narrativa evoluir.

---

Conforme começamos a utilizar IA de forma mais intensa no desenvolvimento, percebemos que os desafios não estavam apenas em como gerar código mais rápido, mas em como melhorar o processo de engenharia como um todo.

A partir disso, comecei a experimentar e consolidar alguns princípios e um workflow para utilizar IA também nas etapas de entendimento, Discovery, planejamento, implementação, entrega e preservação de conhecimento.

Estou compartilhando essa v1.0 ainda como uma proposta em evolução, e não como uma metodologia pronta.

Por isso, gostaria muito de ouvir experiências de outras pessoas que estejam passando por algo parecido:

- Como vocês estão estruturando o uso de IA no processo de desenvolvimento?
- Estão utilizando IA também em Discovery e Planning?
- Criaram algum workflow, Skills ou agentes para isso?
- O que funcionou bem?
- O que não funcionou?
- Que princípios ou etapas vocês acrescentariam?

A ideia é justamente aprender com experiências diferentes e evoluir a metodologia a partir delas.

Abaixo compartilho os fundamentos, o workflow atual, as Skills que já estamos utilizando e algumas das perguntas que ainda estão em aberto.

## Fundamentos

A metodologia está baseada em quatro princípios:

### 1. Reduzir incerteza antes da implementação

O problema precisa ser entendido antes de começar a escrever código.

A IA pode ajudar a identificar lacunas, ambiguidades, riscos, dependências e impactos durante o Discovery.

### 2. Planejamento precede execução

Depois de entender o problema, é necessário transformar esse entendimento em uma estratégia técnica.

A IA passa a trabalhar a partir de contexto e decisões, e não apenas de uma instrução isolada para implementar algo.

### 3. IA como amplificador da engenharia

A IA pode investigar, analisar, estruturar informações, gerar artefatos e executar parte significativa do trabalho.

Mas a responsabilidade pelas decisões, regras de negócio, riscos e qualidade continua sendo humana.

### 4. Conhecimento também faz parte da entrega

Código funcionando não é o único resultado produzido por uma atividade.

Decisões arquiteturais, documentação técnica e o contexto da entrega também precisam ser preservados.

## O workflow atual

Esses fundamentos deram origem ao workflow que estamos experimentando:

**Qualificar → Descobrir → Planejar → Implementar → Entregar → Preservar conhecimento**

Hoje, esse processo é apoiado por um conjunto de Skills que já utilizamos:

- **problem-qualify** — avalia a qualidade da entrada antes do Discovery;
- **discovery** — estrutura o entendimento inicial da demanda;
- **plan** — transforma o entendimento em planejamento técnico;
- **adr-simplificado** — registra decisões arquiteturais;
- **delivery-summary** — consolida informações da entrega;
- **document-technical-reference** — mantém referências técnicas para reutilização e onboarding.

O ponto importante é que essas Skills não são o objetivo da metodologia.

Elas são mecanismos para operacionalizar seus princípios.

## O que ainda não sabemos

A versão inicial dessa metodologia também deixa algumas perguntas em aberto.

### Como medir o ganho real?

Mais uso de IA ou mais tokens não significam necessariamente maior produtividade. Precisamos avaliar impacto, qualidade, retrabalho e tempo de ciclo.

### Qual o nível adequado de autonomia da IA?

Precisamos entender quais atividades podem ser delegadas e onde devem existir pontos de validação humana.

### Como garantir qualidade sem aumentar excessivamente o processo?

Adicionar etapas e automações só faz sentido se elas reduzirem incerteza e aumentarem o resultado, e não se simplesmente criarem burocracia.

### Como manter o conhecimento atualizado?

Documentação, ADRs e referências técnicas também precisam evoluir junto com o código.

Essas perguntas não representam falhas da metodologia.

São parte da própria experimentação.

## E o que muda?

A principal mudança não é simplesmente adicionar IA ao desenvolvimento.

É começar a tratar a IA como parte do workflow de engenharia.

Em vez de:

**Demanda → Código → Teste → Entrega**

buscamos construir um fluxo em que a IA também ajuda a:

**qualificar → entender → planejar → executar → registrar**

Isso desloca parte do esforço para antes da implementação e permite que o engenheiro concentre mais energia em entendimento, estratégia, decisão, validação e qualidade.

Não é uma metodologia fechada nem uma conclusão. É a primeira consolidação de algo que nasceu de problemas reais da equipe e que continuará evoluindo conforme aprendemos com sua aplicação.

---

**Engenharia Aumentada** é usar IA para ampliar a capacidade da engenharia — não apenas para acelerar a escrita de código.
