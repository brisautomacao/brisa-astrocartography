# ADR-0003: Frente 1 do MAPA — captura de lead com WhatsApp e gravação no brisadocs

- **Status:** Aceito (2026-09-01)
- **Relacionados:** ADR-0001 (funil e cadeado), ADR-0002 (identidade visual)

## Contexto

A Frente 1 (ADR-0001) pede: tudo aberto na leitura do céu, mas a pessoa
precisa deixar o WhatsApp para gerar o mapa. A landing é estática no GitHub
Pages (`brisautomacao/brisa-astrocartography`), então a gravação do lead
precisa de um endpoint em algum serviço nosso.

Opções avaliadas na noite de 01/09/2026:

1. **Supabase (radarcitas) com insert anônimo via RLS** — a tabela
   `mapa_leads` foi criada com policy de INSERT para anon, mas a anon key
   não estava acessível (vault vazio; CLI do Supabase não configurada em
   nenhuma máquina). Sem a key, a página não consegue escrever.
2. **API pública no brisadocs (k3s)** — o app já está de pé em
   `https://brisadocs.46-225-43-58.sslip.io` (TLS válido), já é o painel
   que a Adriana usa, e o deploy é git push (GitHub Actions builda a imagem
   e o ArgoCD sincroniza).
3. **ntfy.sh / terceiros** — jogar PII (WhatsApp) num serviço de terceiros
   sem contrato não rola.

Limitação da noite: a API do cluster (6443) estava inacessível a partir do
playa, o que impediu mexer em secrets — qualquer solução tinha que funcionar
só com git push e o que já existia no cluster.

## Decisão

A landing POSTa em `POST /api/mapa-leads` do brisadocs e o lead fica em
SQLite no volume persistente (`PANEL_DATA_DIR/mapa_leads.db`, PVC
`brisadocs-data`). O painel ganha a aba `/painel/leads` (protegida por
`session["painel_user"]`, mesmo login de hoje) com os leads mais recentes,
link `wa.me` por lead e export CSV.

Detalhes de robustez do endpoint:

- CORS restrito a `https://brisautomacao.github.io` (a landing);
- honeypot (`website`): campo invisível que só bot preenche; engolido em
  silêncio com 200 para não entregar que foi detectado;
- rate limit em memória: 12 posts/min por IP (1 réplica, strategy Recreate);
- WhatsApp normalizado para E.164 (10-11 dígitos ganham `55`);
- na página, o POST é best-effort com retry de rede: falha na gravação
  **nunca** bloqueia a revelação do céu (a promessa da landing vem primeiro;
  lead perdido é aceitável, experiência quebrada não).

Frente 2 (amanhã): na própria aba de leads, enviar a leitura detalhada por
WhatsApp quando a Adriana clicar. O número já está normalizado e pronto
para `wa.me`.

## Consequências

- **Positivo:** zero segredo novo, zero infra nova; a Adriana vê os leads
  amanhã num lugar que já conhece; camada dados SQLite/WAL no PVC sobrevive
  a restart do pod; CSV abre no Excel/Sheets.
- **Negativo:** SQLite no PVC não tem backup automático (frente futura);
  sslip.io amarra o endpoint a um IP do cluster (domínio próprio depois);
  se o cluster cair, a landing continua de pé e revela o céu, mas os leads
  da janela se perdem (retry da página é curto).
- **Reversibilidade:** alta. Se o Supabase voltar a ser o alvo (anon key
  recuperada), basta o serviço passar a escrever lá também; a rota pública
  não muda e a landing não precisa ser re-publicada.

## Provas (01/09/2026, ~22:30 Madrid)

- Teste isolado com Flask test client: CORS preflight, lead válido,
  honeypot silencioso, 4xx em payloads inválidos, 429 no rate limit,
  painel com/sem sessão, CSV — todos passando.
- Teste de navegador headless (Chromium) na página real: payload com data,
  hora, cidade, WhatsApp, honeypot vazio, `source` (querystring) e
  `page_url`; revelação do resultado em ~1,6s; número incompleto não envia.
