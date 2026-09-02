# ADR-0009: Link rastreado por pessoa entre o direct do Instagram e a landing

- **Status:** Aceito
- **Data:** 2026-09-02
- **Decisores:** Adriana (produto), Claude (implementação)
- **Complementa:** ADR-0007 (telemetria anônima) e ADR-0008 (mapa sem contato)

## Contexto

Depois do ADR-0008, a Adriana perguntou se dava pra ligar "gerou o mapa e não
deixou contato" a quem a pessoa é no Instagram — ela pediu essa métrica.
Hoje não dá: o link que o bot do brisadocs manda no direct é sempre o mesmo
para qualquer pessoa que toque no chip (`rule["link_url"]` cru), então nenhum
identificador chega até aqui. `VISIT` (ADR-0007) só costura os eventos *da
mesma visita*, não diz quem mandou a pessoa.

## Decisão

O brisadocs passa a gerar um link único por pessoa (`/r/<token>`) em vez da
URL crua, e o token viaja até aqui por `?t=` na query string:

1. Ao carregar, `final.html`/`final-v2.html` leem `?t=` uma vez e guardam em
   `const TOKEN`. Ausente na maioria das visitas (só existe pra quem veio de
   um link do direct) — nesse caso é string vazia, sem quebrar nada.
2. `TOKEN` viaja junto em dois lugares que já mandam PII, por decisão do
   ADR-0008: `sendChart()` (`POST /api/mapa-charts`) e o envio do WhatsApp
   (`POST /api/mapa-leads`, só em `final.html` — a v2 não captura WhatsApp).
3. **Não vai em `track()`/`POST /api/mapa-events`.** Esse contrato é "sem PII"
   (ADR-0007) e continua assim — a resolução do token pra identidade só
   acontece do lado do brisadocs, e só nas tabelas que já guardam PII.
4. O brisadocs decide o que fazer com o token (resolver pra conta+IGSID,
   ignorar se inválido); esta página só repassa o que veio na URL.

## Alternativas consideradas

- **Gerar o token aqui na landing** (ex.: a partir do `referrer`): descartado
  porque `document.referrer` só dá o host (`l.instagram.com`), nunca a pessoa
  — a identidade só existe do lado do brisadocs, que é quem manda a mensagem.
- **Reaproveitar `VISIT` como identificador**: `VISIT` é aleatório por visita,
  gerado no navegador, sem relação nenhuma com quem mandou o link. Serviria
  pra "esta pessoa fez X e depois Y", nunca pra "quem é esta pessoa".

## Consequências

- (+) A Adriana enxerga, na tabela "geraram o mapa e não deixaram contato" do
  `/painel/leads` do brisadocs, quem veio do Instagram — quando o kanban já
  resolveu o perfil, até com `@usuario`.
- (+) Mudança mínima aqui: uma linha lendo a URL, um campo a mais em dois
  `fetch` que já existiam. Toda a complexidade nova (tabela, geração e
  resolução do token, rota de redirecionamento) fica do lado do brisadocs.
- (−) Um link do tipo `/r/<token>` não é legível como a URL final era — se
  precisar debugar manualmente, dá pra colar no navegador (ele redireciona) mas
  não dá pra saber pra onde vai só de olhar.
- (−) Cobre só quem chegou por um link **gerado depois desta mudança**. Não
  há como retroagir identidade a visitas antigas.
