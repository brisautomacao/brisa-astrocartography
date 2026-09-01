# ADR-0001: Landing page do funil MAPA (captura de lead + mapa astral detalhado com cadeado)

- **Status**: Proposto
- **Data**: 2026-09-01
- **Decisor(es)**: Adriana Höher (estratégia de vendas e marca) + Hermes (arquitetura)

---

## Contexto

A Dona Celeste já gera tráfego diário no Instagram, e já existem páginas públicas que calculam
mapa astral no navegador. O que ainda não existe é o **elo entre as duas coisas**: a pessoa comenta
MAPA, mas os dados dela nunca chegam até nós de forma organizada.

### Inventário do que já existe (set/2026)

**1. Motor de tráfego (repo `youtuber`, privado)**
- Persona Dona Celeste definida em `references/dona-celeste-persona.md` (~68 anos, astróloga
  brasileira, voz de "vó astral").
- Reels diários de horóscopo por signo, gerados e publicados por cron no @brisaastral.ai.
- Fechamento travado pelo ADR-0011 (youtuber): *"Segue a vó aqui embaixo e comenta MAPA que te mando
  seu mapa astral. Que os astros te acompanhem!"*, com outro fixo em `assets/outro_dona_celeste.mp4`.
- Todo dia, novos comentários "MAPA" entram e são a matéria-prima do funil.

**2. Produto público (repo `brisa-astrocartography`, público, GitHub Pages)**
- URL: https://brisautomacao.github.io/brisa-astrocartography/
- `docs/mapa-astral.html` (805 linhas): calcula o mapa natal **inteiro no navegador**
  (Astronomy Engine JS): 11 planetas, 12 casas, aspectos, tríade Sol/Lua/ASC, banco de 144
  interpretações signo×casa, tudo em pt-BR. Formulário atual: data, hora, cidade (autocomplete).
- `docs/calculadora.html` (1.135 linhas): mapa astrocártográfico + ranking de 169 cidades com
  filtros por área de vida (amor, dinheiro, carreira, família, desafio).
- `docs/heatmap.html` (527 linhas): mapa de calor mundial de zonas favoráveis.
- **Zero captura de lead**: o que a pessoa digita nunca sai do navegador dela.

**3. Painel de atendimento (repo `brisadocs`, privado, deploy no cluster)**
- Painel multi-conta de Instagram com: regras de comentário por palavra-chave
  (`{keyword, reply, dm}`), agente de IA com tom/objetivo configuráveis, fila de mensagens,
  takeover humano, aba de criação de conteúdo.
- Webhook do Instagram já funciona: comentário com a palavra-chave dispara resposta pública + DM.
- Storage: JSON em volume persistente (`PANEL_DATA_DIR`). Sem SQL (decisão do projeto brisadocs).
- Ingress público com rate limit (10 req/s): `brisadocs.46-225-43-58.sslip.io`.

### O funil hoje (com os elos quebrados)

```
Reel diário (Dona Celeste)
  → pessoa comenta MAPA                     [OK, roda todo dia]
  → regra do painel responde e manda DM     [OK, mecanismo existe]
  → ??? coleta de dados de nascimento      [QUEBRADO: conversa manual no DM]
  → ??? geração do mapa astral              [QUEBRADO: páginas existem mas não são o destino do link]
  → ??? lead organizado no painel           [NÃO EXISTE: não há entidade "lead" com dados]
  → ??? venda da leitura                    [NÃO EXISTE: sem funil, sem follow-up]
```

## Problemas Identificados

1. **Captura inexistente**: as páginas do brisa-astrocartography calculam tudo client-side e não
   enviam nada. A promessa do CTA ("te mando seu mapa astral") é cumprida de forma manual ou não
   é cumprida.
2. **Atrito mortal no DM**: pedir data, hora e cidade de nascimento numa conversa com bot é onde
   leads esfriam. Cada pergunta adicional derruba a conversão.
3. **Sem entidade de lead**: o painel trata conversas, não leads. Não existe registro com
   dados de nascimento, signo, origem (qual reel), status e histórico.
4. **Sem atribuição**: não dá para saber qual reel/signo gera mais leads, porque não há UTM.
5. **Sem mondigitalização**: não há degrau de oferta (nada grátis com cadeado, nada pago,
   nada de alto valor com humano).

## Estratégia de vendas (a visão da estrategista)

Princípios que guiam as decisões técnicas abaixo:

1. **Valor antes do dado, dado antes da venda** (escada de reciprocidade):
   - A pessoa dá 3 campos (data, hora, cidade) e recebe valor real imediato: a tríade
     Sol/Lua/ASC interpretada, na hora, sem cadastro.
   - Depois vê que o mapa COMPLETO já está pronto na tela dela, com 11 planetas, 12 casas e
     aspectos, mas embaixo de um **cadeado com blur**. Para liberar, deixa nome + WhatsApp.
   - O WhatsApp é a moeda do MVP 1. Não cobramos dinheiro ainda: cobramos contato, que vale mais
     (permite venda humana de leitura completa, ticket alto).
2. **Curiosity gap no cadeado**: o conteúdo bloqueado é VISÍVEL (cards reais desfocados, não uma
   tela genérica de "compre"). A pessoa vê que existe e quer o que já é dela por direito cósmico.
3. **Velocidade de resposta**: lead que comentou MAPA está quente por minutos, não dias. A janela
   de ouro de abordagem no painel é de 5 minutos (benchmark de inside sales: responder em 5 min
   multiplica conversão por ~8x vs 1h).
4. **A hora de nascimento é um qualificador**: quem sabe a hora exata tem interesse alto. Quem
   marca "não sei a hora" vira conversa ("a vó te ajuda a descobrir"), não descarte.
5. **Voz única**: toda comunicação (DM, landing, follow-up) na voz da Dona Celeste, carinhosa e
   com autoridade de vó astróloga. Nada de tom corporativo.
6. **Atribuição por origem**: cada link carrega UTM (reel, signo, data). O painel passa a mostrar
   qual conteúdo produz lead, e a produção de reels se otimiza por dado, não por feeling.

### Tratamento do lead no painel (brisadocs)

Pipeline de status: `novo → contato feito → conversando → agendado → cliente → perdido`
(+ `sem hora de nascimento` como sub-etapa de qualificação).

- **Abordagem inicial (janela de 5 min)**: DM no mesmo canal de origem, voz Dona Celeste,
  referenciando o próprio mapa dela: "Filho, seu mapa chegou aqui... aquele Leão de 12/08 tem
  uma Lua em Peixes que explica muita coisa, viu?"
- **Scoring simples (0 a 100)**: +30 WhatsApp válido, +20 sabe a hora de nascimento, +15 veio de
  comentário MAPA (vs link da bio), +10 preencheu até o fim sem erro, +10 reengajou em 72h.
  Hot ≥ 60, morno 30 a 59, frio < 30.
- **Cadência de follow-up**: 24h (lembrete leve), 72h (conteúdo do signo dela), 7 dias (oferta
  da leitura humana com bônus expirável). Para no primeiro resposta.
- **Oferta final (MVP 1 a 2)**: leitura humana de mapa astral completo com a Brisa/Dona Celeste
  (ticket alto, agendamento via WhatsApp). Em MVP 3 entra o desbloqueio pago self-service.

## Decisão

Construir a landing page do funil MAPA como evolução direta do `mapa-astral.html` existente,
hospedada no GitHub Pages do brisa-astrocartography, com captura de lead via um endpoint público
novo no app brisadocs (Flask, já exposto pelo ingress) que cria a entidade "lead" no painel,
e com o cadeado resolvido client-side (libera o conteúdo detalhado após o POST do contato ter
sucesso). Fatiar em MVPs independentes, sendo o MVP 1 exatamente: landing + tríade grátis +
mapa detalhado com cadeado + lead caindo no painel.

## Opções Consideradas

### Opção A: Status quo (páginas atuais, captura manual via DM)

**Prós:**
- Zero trabalho.

**Contras:**
- Funil continua quebrado; a promessa do CTA depende de trabalho manual por lead.
- Sem dados, sem atribuição, sem escala. Mataria o funil quando o volume de comentários subir.

**Veredito:** Insustentável.

### Opção B: Landing estática + desbloqueio por WhatsApp (deep link wa.me)

A landing calcula e mostra a tríade; o cadeado abre somente quando a pessoa manda um WhatsApp
(texto pré-preenchido com nome e dados). O lead "chega" como conversa no WhatsApp da Brisa.

**Prós:**
- Zero backend novo: não mexe no brisadocs para capturar.
- Lead já nasce num canal de conversa (WhatsApp), que é onde a venda acontece.
- Cai no bot de WhatsApp que já roda no brisadocs.

**Contras:**
- Lead vira "conversa", não "registro": dados de nascimento ficam soltos no texto da mensagem.
- Dependência da ação manual da pessoa (abrir WhatsApp, enviar) derruba conversão do unlock.
- Sem POST estruturado, o scoring e a atribuição por UTM ficam cegos.

**Veredito:** Aceitável como fallback degradado, não como solução principal.

### Opção C: Landing estática (GitHub Pages) + endpoint público `/api/leads` no brisadocs ⭐

**Prós:**
- Aproveita 100% do ativo existente: a página já calcula o mapa inteiro client-side; o painel
  já está no ar com ingress público e rate limit (10 req/s, suficiente para o volume do funil).
- Lead estruturado no painel: dados de nascimento, signo, origem, UTM, status, scoring.
- O unlock é imediato e automático (o cadeado abre assim que o POST retorna 200), sem passo
  manual: a melhor conversão de unlock.
- Custo de infra zero (GitHub Pages + cluster que já roda).
- Liga o funil ponta a ponta: comentário MAPA → DM com link → landing → lead no painel.

**Contras:**
- Exige CORS configurado no endpoint (origem `brisautomacao.github.io`).
- Endpoint público precisa de anti-spam básico (honeypot, rate limit por IP, validação).
- Adiciona uma responsabilidade nova ao brisadocs (entidade lead), que hoje só trata conversas.

**Veredito:** **Recomendado.**

### Opção D: Formulário SaaS (Formspree, Typeform, Google Forms)

**Prós:**
- MVP em horas, sem código de backend.

**Contras:**
- Dados ficam fora do painel: o tratamento do lead (que é requisito do negócio) teria que ser
  manual ou via integração frágil.
- Custo mensal, quebra de marca (URL e visual terceiros), fricção a mais para a pessoa.
- Zero atribuição UTM estruturada.

**Veredito:** Não aplicável (contraria o requisito central: tratar o lead NO painel).

### Opção E: Front fala direto com Supabase (anon key + RLS)

**Prós:**
- Escala de dados desde o dia 1, queries ricas no painel.

**Contras:**
- brisadocs decidiu não usar SQL (Google Sheets/JSON). Introduzir Supabase só para leads cria um
  segundo store de verdade divergente do painel.
- Over-engineering para o volume atual (dezenas de leads/dia no máximo no início).

**Veredito:** Prematuro. Reavaliar quando passar de ~1.000 leads/dia (ver Consequências).

## Comparação Resumida

| Critério | A (status quo) | B (wa.me) | C (endpoint brisadocs) | D (SaaS) | E (Supabase) |
|---|---|---|---|---|---|
| Lead estruturado no painel | ❌ | ❌ | ✅ | ❌ | ⚠️ (fora do painel) |
| Tempo até MVP 1 | 0 | ~2 dias | ~4 dias | ~1 dia | ~7 dias |
| Conversão do unlock | n/a | ⚠️ (passo manual) | ✅ (automático) | ⚠️ | ✅ |
| Custo infra | 0 | 0 | 0 | R$/mês | grátis até tier |
| Atribuição por reel/signo (UTM) | ❌ | ❌ | ✅ | ❌ | ✅ |
| Manutenção/agente AI consegue evoluir | n/a | ⚠️ | ✅ (Flask simples) | ❌ | ⚠️ |
| Escala futura | ❌ | ⚠️ | ⚠️ (JSON) | ⚠️ | ✅ |

## Fatias e MVPs

### Fatia 0 (já existe hoje)

Reels diários com CTA travado, páginas de mapa no ar, painel com regras de comentário.
**Gap**: nada disso está conectado.

### MVP 1: Landing + cadeado + lead no painel ⭐ (entrega pedida)

A pessoa coloca os dados de nascimento, recebe valor grátis imediato, e o mapa detalhado
fica atrás de um cadeado que abre quando ela deixa nome + WhatsApp (que chegam ao painel).

**Página nova `docs/mapa.html`** (evolução do `mapa-astral.html`, mantendo o design premium
escuro validado, com a voz visual da Dona Celeste):

1. **Hero**: "Descubra o que os astros dizem sobre você" + subtítulo na voz da vó + prova social
   (contador de mapas gerados, ainda que aproximado no início).
2. **Formulário de nascimento (3 campos)**: data, hora (com a opção "não sei a hora", que segue
   com mapa sem casas e vira conversa de qualificação), cidade com autocomplete (Nominatim, já
   usado na calculadora).
3. **Tríade grátis, instantânea**: Sol, Lua e ASC com interpretação completa (valor real, não
   resumido) + parágrafo conectando os três.
4. **Zona bloqueada (o cadeado)**: os cards REAIS do mapa dela (11 planetas, 12 casas, aspectos)
   renderizados embaixo de blur + ícone de cadeado + copy: "Seu mapa completo já está pronto.
   Libera aí embaixo que a vó te mostra tudo." Botão: **"Liberar meu mapa completo"**.
5. **Formulário de contato (2 campos + 1 checkbox)**: nome, WhatsApp (com máscara), checkbox de
   consentimento LGPD ("aceito receber contato da Dona Celeste sobre astrologia").
6. **POST `/api/leads` no brisadocs** com o lead completo (incluindo um resumo do mapa calculado
   client-side: Sol, Lua, ASC, elemento dominante). Resposta 200 remove o blur e rola a página
   até o conteúdo liberado, com uma mensagem da vó.
7. **CTA secundário pós-unlock**: "Quer entender a fundo com a vó? Chama no WhatsApp" (deep link
   wa.me com contexto do signo). Venda humana como degrau seguinte.

**Endpoint `/api/leads` (brisadocs, app novo blueprint público)**:
- `POST` JSON, CORS restrito à origem do GitHub Pages, honeypot, validação de telefone,
  rate limit por IP (o ingress já limita 10 req/s global; adicionar limite por IP no app).
- Persiste no `PANEL_DATA_DIR` (JSON, mesmo padrão de `panel_config`/`tenant_store`):

```json
{
  "id": "uuid",
  "created_at": "2026-09-01T19:00:00Z",
  "source": "ig_comment",
  "utm": {"reel": "leao_20260901", "medium": "dm"},
  "ig_username": "comentadora_exemplo",
  "name": "Maria",
  "whatsapp": "+5551999999999",
  "birth": {"date": "1992-03-19", "time": "12:00", "time_unknown": false,
             "city": "Porto Alegre", "lat": -30.03, "lon": -51.23},
  "chart_teaser": {"sun": "Peixes", "moon": "Leão", "asc": "Gêmeos",
                    "dominant_element": "Água"},
  "status": "novo",
  "score": 30,
  "history": []
}
```

**Regra MAPA no painel**: configurar `comment_rules` da conta @brisaastral.ai com
keyword `MAPA` (case-insensitive, aceita variações tipo "mapa 🔮"), resposta pública curta da vó
e DM com o link da landing carregando UTM + username: `.../mapa.html?reel=leao_20260901&ig=comentadora_exemplo`.

**Definition of Done do MVP 1**:
- Pessoas reais comentando MAPA recebem DM com link e completam o fluxo até o unlock.
- O lead aparece no painel (mesmo que em lista JSON crua) com dados completos + origem.
- Teste de mesa: lead sem hora de nascimento completa o fluxo (modo degradado).
- PageSpeed mobile ≥ 90 no Lighthouse (o público é 100% mobile via Instagram).

### MVP 2: Tratamento de lead no painel

- Aba "Leads" no console: lista com signo solar, origem (reel), status pipeline, scoring.
- Ficha do lead: dados de nascimento, teaser do mapa, histórico, botão de takeover da conversa.
- Scripts de abordagem por elemento/signo na voz da vó (rascunhos prontos para o agente de IA).
- Alerta de lead quente (comentou e preencheu em menos de 10 min) para abordagem na janela de 5 min.
- Dashboard mínimo: comentários MAPA por dia, taxa DM→landing, taxa landing→unlock, leads por reel.

### MVP 3: Monetização self-service

- Desbloqueio pago opcional (ex.: mapa premium em PDF gerado na hora, R$ 19 a 29) além do
  desbloqueio por WhatsApp. Gateway a decidir em ADR próprio no brisadocs (referência de
  comparação Asaas/Stripe/Mercado Pago/Pagar.me já mapeada).
- Upsell de astrocartografia: a `calculadora.html` e o `heatmap.html` viram segundo ímã
  ("onde morar melhora pra você?"), mesma mecânica de cadeado, mesmo lead enriquecido.

### MVP 4: Escala

- Cadência automatizada de follow-up (24h/72h/7d) via agente de IA do painel.
- Retargeting/pixel quando houver tráfego pago.
- Migração do store de leads se o volume justificar (ver Consequências).

## Plano de Migração

**Fase 1, Fundação (2 dias)**
- Blueprint `/api/leads` no brisadocs + CORS + validação + testes.
- Publicar `mapa.html` derivado do `mapa-astral.html` (form de contato + cadeado + POST).
- Apontar a regra MAPA do painel para a landing com UTM.

**Fase 2, Validação com tráfego real (3 dias)**
- Rodar 2 a 3 reels apontando para o funil, medir DM→landing→unlock.
- Ajustar copy do cadeado e da DM conforme a taxa real (hipótese inicial a calibrar:
  ≥ 30% de DM→landing e ≥ 50% de landing→unlock; números de referência, não spec).

**Fase 3, Painel do lead (MVP 2, ~1 semana)**
- Aba Leads, pipeline de status, scripts, alerta de lead quente.

**Fase 4, Dinheiro (MVP 3, pós-validação)**
- ADR de gateway de pagamento + PDF premium + upsell astrocártográfico.

## Critérios de Aceitação

- Nenhum dado pessoal é coletado sem o checkbox de consentimento LGPD visível.
- O cadeado abre SOMENTE após 200 do POST (não antes, não sem rede, com mensagem de erro clara
  e retry).
- Lead persistido contém: dados de nascimento, teaser do mapa, origem/UTM, contato, timestamp.
- O funil funciona inteiro em mobile Chrome e Safari via Instagram in-app browser.
- O endpoint rejeita spam básico (honeypot preenchido, telefone inválido, IP acima do limite).
- A página não regrediu em performance: Lighthouse mobile ≥ 90.

## Consequências

### Positivas
- O CTA travado do ADR-0011 finalmente tem um destino de valor: a promessa "te mando seu mapa
  astral" é cumprida em self-service, na voz da vó, sem trabalho manual por lead.
- O painel vira a central do funil (como é o objetivo do negócio): lead nasce estruturado, com
  atribuição por reel, pronto para tratamento e venda.
- Custo marginal zero por lead (tudo client-side + infra que já roda).
- Asset 100% próprio (sem lock-in de SaaS de formulário): marca, dados e código ficam conosco.
- A mesma landing serve para bio, stories e tráfego pago no futuro (a UTM diferencia).

### Negativas
- O unlock é client-side: um usuário técnico pode burlar o blur. Mitigação aceita: o valor real
  vendido é a interpretação humana e o serviço, não o HTML. O cadeado é filtro de intenção,
  não de segurança.
- Endpoint público novo no brisadocs = superfície de ataque maior (mitigada com CORS, honeypot,
  rate limit e validação).
- Store em JSON não passa de ~1.000 leads/dia com conforto. Gatilho de migração definido
  (Supabase/SQL em ADR futuro, quando o volume real justificar).
- Mais um sistema no funil = mais um lugar para monitorar (alerta de erro do POST no painel).

### Neutras
- LGPD vira parte do fluxo (checkbox + aviso de uso dos dados; dados de brasileiros, regra LGPD,
  mesmo com a operação em Barcelona).
- O `mapa-astral.html` original permanece como versão "sem funil" para uso interno/da Brisa;
  a landing é uma página irmã, não uma substituição.

## Referências

- ADR-0011 (youtuber): fechamento travado "comenta MAPA" e outro fixo reutilizável.
- Persona: `youtuber/references/dona-celeste-persona.md`.
- Landing base: `docs/mapa-astral.html` (805 linhas, natal chart client-side validado contra
  Swiss Ephemeris).
- Painel: `brisadocs/app/routes/instagram_webhook.py` (regras de comentário), `panel_config.py`
  (formato das regras), `tenant_store.py` (padrão de store JSON do painel).
- Skill astrocartography (Hermes): fórmulas, pitfalls de mobile e o padrão do mapa natal JS
  (`references/natal-chart-js.md`).
