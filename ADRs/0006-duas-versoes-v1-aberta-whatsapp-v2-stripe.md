# ADR-0006: Duas versões do funil — v1 aberta com portão de WhatsApp (país + número completo), v2 monetizada

- **Status:** Aceito
- **Data:** 2026-09-02
- **Decisores:** Adriana (produto), Otávio/Hermes (implementação)

## Contexto

Depois do ADR-0005 (Stripe pendente: a conta ainda não existe e os Payment Links não foram
criados), a Adriana testou a landing publicada e trouxe quatro pontos em 02/09/2026:

1. **Cabeçalho do resultado**: o trio "o que o céu mostrou" + "Três chaves do seu céu" +
   "O essencial sai agora. O resto é conversa." virou ruído. Pedido: só **"a sua essência"** +
   **"O essencial sai agora. O resto é conversa."**
2. **Canoas não aparecia** na busca de cidade (a cidade dela). Investigação: a API do
   Open-Meteo **retorna Canoas/RS normalmente** (primeiro resultado). O problema real era UX
   mobile: a lista de sugestões abria embaixo do campo e ficava escondida atrás do teclado.
3. **Estratégia em duas versões**: publicar já uma versão com **todas as infos abertas**
   (sem paywall), e preparar uma **versão dois** com Stripe em todos os botões, blurs e
   cadeados para quando os links existirem.
4. **Portão de WhatsApp de volta** (reversão parcial do ADR-0005): pra liberar o conteúdo,
   a pessoa deve informar o WhatsApp — agora com **seleção de país** e **número completo**.

## Decisão

### v1 (URL oficial: `sketches/final.html`)

- Chaves (Sol/Lua/ASC) + roda + compartilhar seguem abertos.
- Logo depois, um **portão**: formulário de WhatsApp com `<select>` de país (18 países,
  Brasil default) + campo do número completo com DDI à mostra. Validação no cliente:
  Brasil exige DDD + 9 números começando em 9; demais países aceitam 6-12 dígitos.
- Ao enviar: `POST https://brisadocs.46-225-43-58.sslip.io/api/mapa-leads` (API do
  ADR-0003, honeypot incluso). Se a rede falhar, o conteúdo abre mesmo assim (captura é
  best-effort; o céu nunca fica trancado por erro técnico).
- Após o envio: **tudo aberto** — 8 planetas com texto completo (sem blur, sem 🔒),
  8 moradas de casas (antes 3), top 5 aspectos (antes 3) e o convite final para a Dona
  Celeste apontando para o Instagram (sem preço nesta versão).
- `#preview` continua calculando o mapa da Adriana e agora também destrava tudo sem
  disparar lead.

### v2 (`sketches/final-v2.html`)

- Cópia da versão monetizada do ADR-0005: cadeados, blurs e botões com as constantes
  `LINK_COMPLETO` (R$ 9,90) e `LINK_CELESTE` (R$ 29,90) no topo do script.
- Quando os links do Stripe existirem: preencher as constantes, commit/push e trocar de
  lugar com a v1 (v2 passa a ser `final.html`).

### Correção do autocomplete de cidade (as duas versões)

- `scrollIntoView({block:'center'})` no `focus`/`input` do campo: a lista sobe para fora
  de trás do teclado no celular (causa raiz do "Canoas não aparece").
- **Enter agora escolhe a primeira cidade** da lista quando nenhuma está destacada
  (antes o Enter só funcionava com item navegado por setas).

### Backend (brisadocs)

`normalize_whatsapp` passou a aceitar **E.164 internacional**: número COM `+` é validado
como 8-15 dígitos sem assumir Brasil; sem `+` e com 10-11 dígitos continua assumindo 55.
Sem isso, um número da Espanha (+34, 11 dígitos) seria erroneamente prefixado com 55 e
salvo errado. Testes em `tests/test_mapa_lead_whatsapp.py` (14 casos).

## Opções comparadas

1. **Só a v1 aberta, sem gate** — descartado: perde a captura de lead que é a razão de
   ser da Frente 1 (ADR-0003) e a Adriana pediu expressamente o WhatsApp de volta.
2. **Gate por WhatsApp E pagamento juntos na v1** — descartado: sem conta Stripe ativa,
   não há o que cobrar; a v1 precisa funcionar hoje.
3. **Uma página só, com flag** — descartado por ora: dois arquivos estáticos no Pages é
   mais simples de publicar/reverter do que lógica de feature flag em página estática.
4. **Escolhida:** duas páginas (v1 oficial aberta com gate de WhatsApp; v2 monetizada
   pronta para os links).

## Consequências

- **Positivas:** funil no ar hoje com captura de lead internacional; caminho pronto para
  monetizar (v2) sem refazer nada; causa raiz da busca de cidade corrigida.
- **Negativas:** dois arquivos para manter em paralelo até o Stripe existir (mudanças de
  conteúdo/astrologia precisam ser aplicadas nas duas); v1 não captura lead de quem só
  olha as chaves (aceitável: chaves são a isca).
- ADR-0005 fica **Aceito e agendado** (v2); ADR-0003 volta a valer por completo na v1.
