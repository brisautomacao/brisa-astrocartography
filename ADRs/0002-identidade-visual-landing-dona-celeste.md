# ADR-0002: Identidade visual da landing do MAPA (Dona Celeste)

**Status**: Proposto
**Data**: 2026-09-01
**Decisores**: Adriana Höher (direção criativa), Otavio/Comau (implementação)

## Contexto

A ADR-0001 definiu a arquitetura do funil MAPA (landing `docs/mapa.html` com
tríade grátis + mapa completo atrás de cadeado, captura de lead no brisadocs).
Esta ADR trava a **direção de arte** da página, definida pela Adriana em
2026-09-01:

> "Uma página com a Celeste na página, o mapa astral em desenhos, uma página
> simples mas super esotérica e surpreendente. Deve se parecer com ela. E com
> o ambiente que temos hoje nos seus vídeos. Os cadeados devem aparecer uma
> foto pequena da dona celeste dizendo que pode interpretar melhor o que
> significa."

### Especificação canônica da Dona Celeste (fonte: Adriana, 2026-09-01)

Esta é a descrição mestra do personagem para qualquer arte da página
(verbatim, PT e EN; a EN serve para ferramentas de geração de imagem):

**PT**: Retrato de uma mulher brasileira de 68 anos, pele morena escura com
subtons dourados, rugas naturais de sorriso, pele viçosa. Cabelo prateado
muito longo e volumoso, em ondas em cascata que passam do peito, repartido
ligeiramente fora do centro, com uma mecha branca brilhante na raiz. Sem
óculos. Grandes argolas douradas e colares finos de ouro sobrepostos. Blusa
elegante em roxo profundo com brilho sutil. Sentada em uma sala de astrologia
iluminada à luz de velas, com cristais, livros antigos e um telescópio de
latão desfocados ao fundo. Maquiagem natural e quente, sombra bronze, lábio
rosé, sorriso suave, olhar caloroso direto para a câmera, enquadramento
vertical 9:16.

**TOM**: caloroso, sereno, com autoridade de quem já viu tudo. Ternura,
sabedoria e um brilho de malícia boa. Nunca determinista: fala de reflexões,
orientações e possibilidades.

**EN** (para image generation): Portrait of a warm 68-year-old deep
warm-brown-skinned Brazilian woman with golden undertones, natural laugh
lines, healthy vibrant skin, very long voluminous silver-gray hair in
cascading S-shaped waves reaching past her chest, parted slightly off-center
with a bright white streak at the hairline, no glasses, large gold hoop
earrings, layered thin gold necklaces, elegant deep-purple top with subtle
shimmer, seated in a warm candlelit astrology room with crystals, old books
and a brass telescope softly blurred behind her, natural warm makeup with
bronzy eyeshadow and rose lip, gentle closed-mouth smile, warm direct eye
contact, vertical 9:16 framing.

### Assets existentes

- `youtuber/assets/anchor.jpg` (+ `anchor_mirror.jpg`): retrato travado do
  avatar usado nos vídeos. **Confere com a especificação canônica** (cabelo
  prateado longo, sem óculos, blusa roxa com brilho, argolas douradas, sala à
  luz de velas). É a fonte oficial da foto da Celeste na página.
- `youtuber/assets/outro_dona_celeste.mp4`: vídeo de encerramento; frames
  analisados para extração de paleta (livro aberto com carta celeste,
  cristais, velas, telescópio de latão).
- **Drift conhecido**: `youtuber/references/dona-celeste-persona.md`
  (seção Appearance) ainda descreve a versão antiga (óculos dourados, cabelo
  castanho pixie). A especificação canônica acima prevalece; sincronizar o
  arquivo do youtuber é tarefa separada (repo de Otavio).

### Paleta extraída dos assets reais (anchor.jpg + frames do outro)

| Uso | Cor | Hex |
|---|---|---|
| Fundo profundo (sala/sombra) | marrom-roxo quase preto | `#2B1B17`, `#2E1E2A` |
| Roxo profundo (blusa, molduras) | berinjela | `#4A2040`, `#5B244D` |
| Dourado/latão (joias, telescópio, ícones, CTA) | ouro velho | `#C89F50`, `#A47F5D` |
| Luz de vela (realces, glow) | âmbar | `#F7B567` |
| Creme (texto sobre escuro, papel do mapa) | pergaminho | `#E9D5BD`, `#F5F5DC` |
| Prata (cabelo, detalhes frios) | cinza claro | `#9D9D9D`, `#C8C9CB` |
| Lilás cristal (acentos sutis) | ametista | `#5E3A53`, `#ADB5BD` |

## Problemas

1. A página atual (`mapa-astral.html`, 805 linhas) tem cara de ferramenta
   técnica (roda de mapa clínica, sem personagem, sem clima). Não gera
   vínculo emocional nem transmite a autoridade da Celeste.
2. O produto é uma leitura humana de alto valor; a página precisa *parecer*
   handmade e esotérica, não um app SaaS.
3. A foto da Celeste existe em qualidade (anchor.jpg) mas não há padrão
   definido de como ela aparece na página, nem no componente cadeado.
4. Gosto da Adriana já mapeado em outros projetos (instagram-posts):
   místico editorial, **rejeitado**: Keith Haring, saturação alta, figuras
   SVG clipart genéricas.

## Opções consideradas

**A. "Mesa da Celeste" (escuro à luz de vela)**: fundo marrom-roxo profundo,
página como se fosse a mesa dela; carta do mapa desenhada à mão em papel
envelhecido; velas e cristais como elementos vivos; dourado nos CTA.
Espelha 1:1 o ambiente dos vídeos.

**B. "Céu profundo" (noite estrelada)**: azul-marinho/preto com
constelações animadas, faixa de estrelas prateadas, mapa em tinta dourada.
Mais "astronomia", menos "sala da vó".

**C. "Editorial esotérico" (claro)**: fundo creme papel, tinta sépia,
dourado, serifa editorial (linha do gosto já aprovado pela Adriana em
posts). Elegante mas se distancia do ambiente candlelit dos vídeos.

## Comparação

- Fidelidade ao pedido ("ambiente dos vídeos"): A máxima, B média, C baixa.
- Legibilidade mobile (público 100% celular via DM): A boa com creme sobre
  escuro (contraste `#E9D5BD` sobre `#2B1B17` ≈ 12:1), B boa, C excelente.
- Diferenciação vs concorrentes de mapa astral (todos clínicos/claros):
  A e B altíssima, C média.
- Surpresa/memorabilidade: A alta (a "sala" é a marca), B alta, C média.
- Risco: A exige craft na textura de papel/vela para não virar "dark mode
  genérico"; mitigado pelos desenhos à mão e pelo retrato real.

## Decisão Final

**Opção A, "Mesa da Celeste"**, com elementos de C na tipografia (serifa
editorial para títulos). Direção de arte:

1. **Ambiente**: fundo escuro candlelit (gradiente radial `#2E1E2A` →
   `#2B1B17`), luz âmbar `#F7B567` como glow suave no topo (vela fora de
   quadro). Micro-interações: leve cintilação de estrelas prata, chama de
   vela animada sutil, poeira dourada flutuando (CSS, sem vídeo pesado).
2. **Tipografia**: Fraunces (títulos, itálico para frases da Celeste) +
   Outfit (corpo/formulário). Creme `#E9D5BD` para texto; nunca branco puro.
3. **A Celeste presente**: hero com o retrato `anchor.jpg` em moldura
   orgânica (bordas irregulares como papel queimado, filete dourado) +
   saudação em primeira pessoa no tom dela (sem se nomear, regra de
   anti-autorreferência dos vídeos): ex. *"Óh, meu bem... deixa eu ver o
   seu céu."*
4. **Mapa astral em desenhos**: carta desenhada à mão (estilo gravura/tinta
   sobre papel envelhecido `#E9D5BD`), planetas como pequenas ilustrações à
   tinta com realce dourado, signos como glifos inkados, linhas de aspecto
   pontilhadas como costura. **Proibido**: roda clínica de software,
   clipart, SVG stock, saturação alta. O desenho é desenhado para a página
   (inline SVG autoral ou ilustração raster otimizada WebP).
5. **Componente cadeado (padrão repetível)**: seções bloqueadas (mapa
   completo, casas, aspectos) aparecem em blur quente; sobre elas, um chip
   circular de ~72px com a foto da Celeste (crop do anchor.jpg, anel
   dourado), ícone de cadeado dourado e microcopy no tom dela, ex.:
   *"Isso aqui eu leio pra você. Quer que eu interprete o que significa?"*
   → botão dourado **"Liberar com a Celeste"** (abre o formulário nome +
   WhatsApp da ADR-0001).
6. **Formulário de nascimento**: campos grandes, mão amiga, fundo de papel;
   feedback esotérico (ao calcular, "acendendo as velas..." com micro-delay
   teatral de 1-2s).
7. **Regras de conteúdo**: nunca determinista (reflexões/orientações), nada
   de promessa de futuro certo; texto da página em pt-BR coloquial da
   persona (meu bem, anota aí).

## Adendo 1 (2026-09-01): cartão compartilhável

A Adriana pediu que a tríade + carta possam ser exportadas para story/feed. Decisão:
- **Como**: canvas 2D desenhado no próprio navegador (`makeCard(W,H)` em `docs/sketches/final.html`), sem backend. Botões "Compartilhar no story" (1080x1920) e "Compartilhar no feed" (1080x1350) usam a **Web Share API com arquivo** (`navigator.share({files})`): abre a folha de compartilhamento do sistema com o Instagram e a imagem anexada. Fallback quando o navegador não suporta (desktop, navegadores de app): mostra o cartão na tela com opção de salvar.
- **Regras de texto (Adriana, 2026-09-01)**: nunca afirmar tempo de experiência que não é real (removido "cinquenta anos"); usar linha inspiradora no lugar. Não repetir palavra "trancado": curiosidade por conteúdo ("os outros moradores do céu", "doze gavetas"), cadeado apenas como ícone discreto.
- **Conteúdo do cartão**: kicker "O MEU CÉU", as três chaves em Fraunces itálico com **"Ascendente" por extenso** e **mini descrição sob cada chave** ("o que faz o seu coração brilhar", "o jeito de sentir e de cuidar", "a porta por onde o mundo te encontra"), a carta em linha dourada com **anel dos 12 signos** (também na carta SVG da página), e uma única linha de CTA: "Faça seu mapa em @brisaastral.ai" (handle em dourado). Sem foto da Celeste e sem instrução de comentar MAPA: decisão da Adriana (2026-09-01) por um cartão mais limpo; a marca continua no subtítulo "lidas por Dona Celeste".
- **Tease do cartão (Adriana, 2026-09-01)**: removido "e tem mais: oito planetas, doze casas e conversas entre estrelas" do story (não fazia sentido para ela); no feed permanece apenas "e isso é só o começo ✨".
- **Por quê**: o lead posta o próprio céu e vira mídia do funil; o CTA fecha o loop reels → MAPA → landing → story do lead → novos comentários MAPA.
- **No MVP 1**: os valores da tríade e as posições dos planetas saem do cálculo real (Swiss Ephemeris), não dos dados de exemplo.

## Plano de Migração

1. Preparar assets: crops circulares da Celeste em 2-3 tamanhos (chip
   72px, hero 9:16), WebP, total da página < 900KB.
2. Sketch de 2-3 variantes da Opção A (ritmo, densidade, hero) publicado
   em `/sketches/` no GitHub Pages para a Adriana escolher pelo celular.
3. Variant escolhido vira `docs/mapa.html` novo (mantendo
   `mapa-astral.html` como legado até a troca do link na regra MAPA).
4. Desenho autoral da carta (planeta a planeta) validado com a Adriana
   antes da implementação final dos cadeados.

## Critérios de Aceitação

- Página carrega < 3s em 4G medíocre (celular público-alvo).
- Reconhecibilidade imediata: print da página lado a lado de um frame do
  vídeo deve parecer "o mesmo mundo" (paleta + retrato + papel).
- Contraste mínimo AA (4.5:1) para todo texto; creme sobre escuro atende.
- Cada seção bloqueada usa o chip Celeste + microcopy + CTA único.
- Zero menção a "vó"/"Dona Celeste" em fala em primeira pessoa na página
  (regra anti-autorreferência); o nome pode aparecer apenas em assinatura
  de caption/rodapé se a Adriana pedir.
- Funciona 100% client-side no GitHub Pages; eventos de analytics via
  POST para o brisadocs (ADR-0001) sem quebrar a página se offline.

## Consequências

**Positivas**: página única no nicho (nenhum concorrente "candlelit");
continuidade perfeita vídeo → link (a pessoa sai da DM e entra na mesma
sala); o chip Celeste humaniza o cadeado e pré-vende a leitura humana.
**Negativas**: craft alto (ilustração autoral, texturas) = mais tempo de
design que um template; páginas escuras com foto exigem otimização de peso
(WebP, tamanhos responsivos); manutenção de mais assets (crops da Celeste).
**Neutras**: a paleta passa a ser referência de marca para futuras peças
(e-mail, WhatsApp Business, páginas de obrigado).
