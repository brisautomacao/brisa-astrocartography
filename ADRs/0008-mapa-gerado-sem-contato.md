# ADR-0008: Guardar o mapa gerado mesmo quando a pessoa não deixa contato

- **Status:** Aceito
- **Data:** 2026-09-02
- **Decisores:** Adriana (produto), Claude (implementação)
- **Complementa:** ADR-0003 (leads) e ADR-0007 (telemetria anônima)

## Contexto

Depois do ADR-0007, a Adriana passou a ver o funil inteiro em número: quantas
visitas entraram, quantas geraram o mapa, quantas chegaram no portão e quantas
deixaram o WhatsApp. O que ela não vê é **quem**.

Hoje os dados de nascimento só são gravados quando a pessoa entrega o WhatsApp
(`/api/mapa-leads`). Quem preenche data, hora e cidade, lê a parte aberta e vai
embora sem deixar o número desaparece por completo: sobra um `chart` anônimo no
contador e mais nada.

Pedido dela, textual: *"quero que a gente guarde as informações de quem preenche
o mapa astral, nem que seja pra alguma chamada de log que seja"*.

## Decisão

Uma tabela separada, `mapa_charts`, no mesmo SQLite do PVC:

1. **`POST /api/mapa-charts`** (público, CORS só pro Pages, honeypot e rate limit
   iguais aos dos leads). A landing chama junto do evento `chart`, uma vez por
   visita (`chartSent`).
2. **Campos**: `birth_date`, `birth_time`, `city`, `visit`, `source`, `page_url`.
   **Sem WhatsApp, sem IP e sem user agent** — de propósito: não houve troca
   nenhuma com essa pessoa, então guardamos o mínimo. Os leads continuam
   gravando IP e UA porque ali existe consentimento na prática (ela entregou o
   número em troca da leitura).
3. **`visit`** é o mesmo id aleatório da telemetria do ADR-0007. Ele costura o
   mapa ao lead se a pessoa voltar e deixar o número, sem precisar de cookie.
4. **Painel**: segunda tabela em `/painel/leads` ("geraram o mapa e não deixaram
   contato") e `/painel/mapas.csv`.
5. **`#preview` não grava nada**, igual à telemetria.
6. **Validação compartilhada**: `_validate_nascimento` passa a ser usada pelo
   lead e pelo mapa. As regras de data, hora e cidade são as mesmas e não podem
   divergir.

## Alternativas consideradas

- **Aceitar lead sem WhatsApp na tabela `mapa_leads`**: menos código, mas
  afrouxaria a validação da API que hoje sustenta a Frente 2 (enviar a leitura
  por WhatsApp) e misturaria quem dá pra contatar com quem não dá.
- **Mandar o nascimento junto do evento `chart`**: seria a menor mudança, mas
  quebraria a promessa do ADR-0007 de que `mapa_events` não guarda PII.

## Consequências

- (+) A Adriana passa a ver o que as pessoas pediram, mesmo quem não converteu:
  volume real, distribuição de cidades e de datas.
- (+) Dá pra medir o portão de verdade — quantos mapas geraram contato e quantos
  não — pessoa a pessoa, não só em contador agregado.
- (−) **Passamos a guardar PII sem contrapartida.** Data, hora e cidade de
  nascimento identificam alguém. O ADR-0007 dizia "sem PII, logo sem banner"; a
  partir daqui isso vale só para `mapa_events`. A landing precisa dizer, em
  texto visível, que o mapa gerado fica guardado. **Pendente.**
- (−) Uma pessoa que gera três mapas (pra ela, pro namorado, pra mãe) vira uma
  linha só: `chartSent` manda o primeiro e ignora os outros. Trocar por "um por
  mapa" é mudar uma linha, mas aí o volume infla e o funil do ADR-0007 deixa de
  bater com esta tabela.
