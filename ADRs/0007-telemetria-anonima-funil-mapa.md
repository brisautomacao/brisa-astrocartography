# ADR-0007: Telemetria anônima do funil do MAPA (contador de passos, sem PII)

- **Status:** Aceito
- **Data:** 2026-09-02
- **Decisores:** Adriana (produto), Otávio/Hermes (implementação)
- **Suplanta:** nada (complementa o ADR-0003)

## Contexto

O ADR-0006 devolveu o funil à ativa (v1 com portão de WhatsApp, v2 com Stripe), mas
a Adriana só enxergava quem deixava o número (aba leads). Quem entrava na página e
saía sem interagir era invisível: GitHub Pages não registra visitas. Adriana pediu
um contador ("como está vigiando os CTR ou quem entra na página?") e aprovou a
proposta de um funil anônimo no próprio brisadocs.

## Decisão

Telemetria **própria, anônima e sem cookies**, no padrão do ADR-0003:

1. **Eventos** (whitelist no serviço):
   - `enter` — página carregou (uma vez por visita)
   - `chart` — gerou o mapa (tocou em "Apontar o telescópio")
   - `gate` — o portão de WhatsApp ficou visível (v1) / toque nos CTAs
   - `unlock` — deixou o WhatsApp e destravou (v1)
   - `pay` — tocou em botão de pagamento (v2)
   - `share` — pediu cartão pro story/feed
2. **Payload sem PII**: tipo do evento, versão da página (`v1`/`v2`), **host** do
   referrer (ex.: `l.instagram.com`) e um id de visita aleatório gerado no
   navegador. Nada de IP, UA ou URL completa. Sem cookies, logo sem banner de
   consentimento (LGPD/GDPR friendly).
3. **API**: `POST /api/mapa-events` no brisadocs, CORS só pro Pages, rate limit
   60/min/IP, honeypot igual ao dos leads. SQLite `mapa_events.db` no mesmo PVC.
4. **Painel**: aba **📊 Funil do MAPA** (`/painel/funil`, atrás do login) com
   últimos 7 dias em resumo, tabela por dia (horário de Madrid) com conversões
   (gerou/entrou e WhatsApp/portão) e o top de origens das visitas.
5. **`#preview` não conta nada**: Adriana espia a página sem sujar os números.
6. **Testes**: `tests/test_mapa_event_service.py` (validação, agregação por dia
   Madrid, origens), carregamento por stubs como o dos leads.

## Consequências

- (+) Adriana passa a ver CTR do funil inteiro (visita → mapa → portão → WhatsApp)
  no mesmo painel dos leads, sem depender de terceiros.
- (+) Host do referrer responde "de onde vieram" (bio do Instagram vs. direto).
- (+) Mesma stack (Flask + SQLite + Pages), zero custo, zero conta nova.
- (−) Contagens não são à prova de adblock (fetch falha = passo perdido) e não há
  deduplicação por usuário, só o id de visita por carregamento. Suficiente pra
  decidir produto; se um dia precisar de precisão de auditoria, avaliar Plausible.
- (−) v1 e v2 contam no mesmo balde, separadas pela coluna `page` (o painel atual
  agrega as duas; separar por versão é só agrupar a mais).
