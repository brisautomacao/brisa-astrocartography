# ADRs do brisa-astrocartography

Registro de decisões de arquitetura e produto do funil de mapa astral da Brisa Astral
(Dona Celeste). Um ADR documenta UMA decisão, com contexto, opções comparadas e justificativa.

## Regras

- Um ADR por decisão, numerado sequencialmente: `NNNN-titulo-curto-em-kebab-case.md`
- Status: `Proposto` → `Aceito` → `Depreciado/Substituído` (ADRs nunca são deletados; substituídos apontam para o sucessor)
- Idioma: português (decisões com participação da Adriana)
- Toda opção de custo traz data de verificação dos preços

## Índice

| ADR | Título | Status |
|---|---|---|
| [0001](0001-landing-page-funil-mapa-astral.md) | Landing page do funil MAPA (captura de lead + mapa astral detalhado com cadeado) | Aceito (Frente 1 entregue 01/09/2026) |
| [0002](0002-identidade-visual-landing-dona-celeste.md) | Identidade visual da landing: Mesa da Celeste (candlelit, paleta dos vídeos, chip Celeste no cadeado) | Aceito |
| [0003](0003-frente1-captura-de-lead-whatsapp-brisadocs.md) | Frente 1: captura de lead com WhatsApp via API pública do brisadocs (SQLite no PVC, aba /painel/leads) | Aceito (suspenso por ADR-0005; reativado na v1 por ADR-0006) |
| [0004](0004-calculo-real-no-navegador-astronomy-engine.md) | Cálculo real do mapa astral no navegador (Astronomy Engine + Open-Meteo geocoding, validação de cidade) | Aceito |
| [0005](0005-monetizacao-stripe-pausa-whatsapp.md) | Monetização com Stripe Payment Links (R$ 9,90 / R$ 29,90), UX para público mais velho, pausa da captura de WhatsApp | Aceito (agendado: aplica-se à v2, ver ADR-0006) |
| [0006](0006-duas-versoes-v1-aberta-whatsapp-v2-stripe.md) | Duas versões do funil: v1 aberta com portão de WhatsApp (país + número completo) e v2 monetizada com Stripe, ambas com blur/cadeados de curiosidade | Aceito |
| [0007](0007-telemetria-anonima-funil-mapa.md) | Telemetria anônima do funil: eventos sem PII no brisadocs (POST /api/mapa-events) e aba 📊 Funil no painel | Aceito |
| [0008](0008-mapa-gerado-sem-contato.md) | Guardar o mapa gerado (nascimento e cidade) mesmo sem contato: POST /api/mapa-charts e segunda tabela em /painel/leads | Aceito |
| [0009](0009-link-rastreado-por-pessoa-do-direct.md) | Link rastreado por pessoa (/r/&lt;token&gt; no brisadocs, ?t= aqui): liga quem gerou o mapa a quem é no Instagram | Aceito |
