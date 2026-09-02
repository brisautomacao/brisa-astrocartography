# ADR-0004: Cálculo real do mapa astral no navegador (Astronomy Engine + geocoding)

- **Status:** Aceito
- **Data:** 02/09/2026
- **Frente:** 1 (funil MAPA)

## Contexto

A `docs/sketches/final.html` (landing do funil) mostrava valores **fixos** para
todo mundo: Sol em Leão, Lua em Escorpião, Ascendente em Peixes, roda desenhada
com posições estáticas. A Adriana identificou o problema no feedback de
02/09/2026 ("você não está calculando o mapa astral real"). O produto promete um
mapa astral; entregar valores de exemplo quebra a confiança e a promessa da
Dona Celeste.

Requisito adicional do mesmo feedback: o campo de cidade precisa sugerir
**cidades que existem** (lista embaixo do campo), para validar a entrada e obter
coordenadas.

## Decisão

Calcular o mapa **no cliente**, sem backend, com:

1. **Efemérides:** Astronomy Engine (`docs/astronomy.min.js`), já usada pelo
   `mapa-astral.html`. Posições aparentes geocêntricas via
   `GeoVector` + `Ecliptic`.
2. **Verificação contra Swiss Ephemeris** (pyswisseph 2.10.03, FLG_MOSEPH) em 3
   casos (Porto Alegre 27/05/1989 11:30 UTC, Barcelona 15/03/1960, Recife
   20/11/1975): erro máximo observado **~0,003°** em Sol, Lua, Mercúrio, Vênus,
   Marte, Júpiter, Saturno, Urano, Netuno, Plutão, Nodo médio, ASC e MC.
3. **ASC/MC:** tempo sideral local (`Astronomy.SiderealTime` + longitude) e
   fórmulas clássicas de atan2 com correção de quadrante (a mesma do
   `mapa-astral.html`, validada aqui contra `swe.houses_ex`).
4. **Fuso horário:** conversão hora local -> UTC pela **timezone IANA da cidade**
   (retornada pelo geocoder) usando `Intl.DateTimeFormat` com iteração
   convergente. Isto corrige o horário de verão (ex.: Brasil até 2019,
   Europa/Madrid no verão). O `mapa-astral.html` usa `lon/15` arredondado, que
   erra fusos políticos; a landing nova não herda esse erro.
5. **Casas:** iguais (Equal House) a partir do ASC; Nodo Lunar médio; aspectos
   com orbes 8°/8°/7°/7°/6° (conj/opos/trig/quadr/sext).
6. **Geocoding e validação de cidade:** Open-Meteo Geocoding API
   (`geocoding-api.open-meteo.com/v1/search`), sem chave e sem custo, `language=pt`,
   resultados ordenados por população no cliente. A cidade **precisa** ser
   escolhida na lista (autocomplete com teclado: setas + Enter); digitar sem
   escolher bloqueia o envio com mensagem amigável.

## Opções consideradas

| Opção | Prós | Contras | Verificado em |
|---|---|---|---|
| API externa de astrologia | Menos código | Chave, custo, latência, dependência | 02/09/2026 |
| Cálculo no cliente (escolhida) | Zero custo/backend, offline após carregar | Código JS na página | 02/09/2026 |
| Cálculo no brisadocs | Centraliza lógica | Backend para página estática, deploy a cada ajuste | 02/09/2026 |

## Consequências

- Cada visitante vê o próprio céu: Sol, Lua, ASC, casas dos planetas e aspectos
  reais alimentam também os teasers bloqueados (a promessa do cadeado é
  verdadeira, o que aumenta a conversão).
- A roda SVG e o cartão de compartilhamento (canvas) são desenhados a partir das
  posições calculadas, com ASC à esquerda.
- Sem hora de nascimento: usa meio-dia local e avisa que o Ascendente pode
  variar (decisão de produto, voz da Celeste).
- `#preview` no fim da URL renderiza o mapa da Adriana (POA 27/05/1989 08:30)
  para conferência rápida: Sol em Gêmeos, Lua em Aquário, ASC em Gêmeos.
