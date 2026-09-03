# ADR-0010: Carrossel do vídeo do dia (Instagram)

**Status**: Aceito
**Data**: 2026-09-03
**Decisores**: Adriana Höher (direção criativa), Hermes (implementação)

## Contexto

Os reels diários da Dona Celeste (@brisaastral.ai) têm copy aprovada no roteiro
do pipeline de vídeo (`output/planned/YYYY-MM-DD_<signo>_roteiro.md`, fechamento
fixo do ADR-0011 do youtuber) que hoje só vive no vídeo. A Adriana pediu um
agente que transforme essa mesma copy em carrossel, com as fotos da Celeste e a
identidade da landing. Libra e Leão foram gerados, revisados por ela no Telegram
e aprovados em 03/09/2026, e ela pediu a fila de um carrossel por signo.

## Decisão

1. **Fonte única de copy**: o roteiro do pipeline de vídeo (mesma copy do reel,
   beats verbatim + fechamento no caption da postagem). Copy de signo sem vídeo
   ainda: `research.py --type horoscope --sign <Signo>` (ASCII), SEMPRE com
   checagem factual contra as posições reais antes de virar slide.
2. **Identidade visual**: herda a ADR-0002 (Mesa da Celeste): fundo roxo-vela
   com brilho âmbar, ouro velho, pergaminho; Fraunces/Outfit/Great Vibes;
   retratos circulares da Celeste (`dona_celeste_master.png`) com margem
   superior folgada para a foto aparecer sempre.
3. **Estrutura (máx 10 páginas)**: capa (só o nome do signo, ex. "Libra") +
   mapa do dia + 1 página por beat + convite ("Comenta MAPA").
4. **Mapa do dia, título "A foto do céu hoje!"**: roda zodiacal no formato
   EXATO do `drawWheel` da landing: anel externo tracejado (dasharray 1 6),
   anel interno, ticks de 30°, glifos serif no aro, pontos dourados no raio 110
   com rótulos ABREV (SOL, LUA, VÊN...) e linhas guia, teia de aspectos
   tracejada âmbar (top 12 por menor orbe: conj/oposição 8°, tríneo/quadratura
   7°, sextil 6°), orientação ASC às 9h anti-horário. Posições reais via
   skyfield (`instagram_carousel.aspectos`). Meta-line com data e fase da Lua.
5. **Tríade com significados**: Sol (a essência de hoje), Lua (a emoção de
   hoje) e a chave ligada ao signo do dia: regente em casa, ou planeta de
   visita quando o regente é o próprio Sol/Lua. Os significados são os versos
   da própria landing, com ênfase âmbar + ✦ na chave que tem mais a ver com o
   signo de hoje.
6. **Planetas nos beats**: beat que cita planeta ganha o disco desenhado dele
   (fase real da Lua para o hemisfério sul; fase de Vênus pela elongação do
   Sol; detalhe por planeta: anel de Saturno, faixas de Júpiter, crateras da
   Lua, manchas de Marte). Vale o PRIMEIRO planeta citado no texto. Sem
   numeração "1 de N" nas páginas (rejeitada pela Adriana).
7. **Ferramenta e fluxo**: agente em
   `~/.hermes/skills/media/content-pipelines/scripts/video-carousel/gen_video_carousel.py`;
   render Playwright 1080×1350; verificação programática de layout (gaps
   positivos entre zonas, sem estouro do canvas); slides enviados à Adriana no
   Telegram um por vez; publicação só após aprovação dela
   (`RealIGClient.post_carousel` com caption + hashtags do `carousel_meta.json`).
8. **Custo reportado**: após cada carrossel criado, a Adriana recebe o custo de
   API estimado (copy via LLM + leituras de QA visual). Render e efemérides
   custam zero: roda, discos e teia de aspectos são SVG locais no playa.

## Consequências

- A fila pode rodar um carrossel por signo em lotes (30 em 30 minutos, pedido
  da Adriana em 03/09/2026, começando por Leão).
- Copy de `research.py` exige checagem factual contra as posições reais: caso
  real do Leão, o research disse "Lua em Gêmeos" quando a Lua ainda estava a
  28° de Touro; corrigido para "a Lua que entra em Gêmeos hoje".
- Custo por carrossel fica na casa de centavos de dólar (sem geração de imagem
  paga; tudo SVG/Playwright local + 1 chamada LLM de copy + QA visual).

## Alternativas rejeitadas

- Pipeline automático de signos (renderer Pillow do instagram_carousel):
  identidade antiga, diferente da landing.
- Template editorial v9 (creme `#DBDAD5` + laranja queimado): direção anterior
  da Adriana, não é a Mesa da Celeste.
- Numeração "1 de N" nas páginas: rejeitada pela Adriana.
- wheel "decorativo" com posições arbitrárias: contradiz o contrato de
  correspondência desenho x texto (o ponto do SOL tem que estar no setor do
  signo do Sol, igual à landing).
