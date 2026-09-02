# ADR-0005: Monetização com Stripe Payment Links (R$ 9,90 / R$ 29,90) e pausa da captura de WhatsApp

- **Status:** Aceito
- **Data:** 02/09/2026
- **Frente:** 1 (funil MAPA)

## Contexto

Feedback da Adriana de 02/09/2026:

1. Nos cadeados e trechos borrados, colocar link de pagamento de **R$ 9,90**
   ("revelação completa").
2. **Cada seção** do mapa deve ter um botão "Revelação completa R$ 9,90".
3. O fechamento vira **"Interpretação com a Dona Celeste" por R$ 29,90**
   (antes: "liberar com a Celeste").
4. Público-alvo são **pessoas mais velhas**: fontes maiores, botões grandes e
   claros.
5. Formulário **sem WhatsApp por enquanto**.

A conta Stripe ainda não existe; o acesso não foi configurado (pergunta da
Adriana: "você tem acesso ao stripe?"). Os botões já estão na página apontando
para constantes únicas, prontas para receber os links.

## Decisão

1. **Stripe Payment Links** como mecanismo de cobrança: dois produtos
   ("Revelação completa" R$ 9,90; "Interpretação com a Dona Celeste" R$ 29,90),
   links criados no painel do Stripe (ou via API com chave restrita, se a
   Adriana preferir que o agente crie) e colados nas constantes
   `LINK_COMPLETO` / `LINK_CELESTE` no topo do `<script>` de `final.html`.
   Enquanto vazias, os botões avisam com um alerta no tom da Celeste.
2. **Botões em toda seção** (3 chaves, roda, planetas, casas, aspectos) + botão
   final claro R$ 29,90. Botões dourados preenchidos, texto escuro, alvo de
   toque grande.
3. **Acessibilidade para pessoas mais velhas:** base 17px, inputs 21px (Fraunces),
   botões com padding generoso, zoom do navegador **não** bloqueado, mensagens de
   erro no tom da personagem.
4. **WhatsApp pausado:** o campo saiu do formulário e o POST de lead foi
   **removido da landing** (a API pública do brisadocs exige
   `whatsapp_obrigatorio`; ADR-0003 fica **parcialmente suspenso**: backend e
   aba /painel/leads continuam no ar e intactos para uso futuro).

## Opções consideradas (pagamento)

| Opção | Prós | Contras | Verificado em |
|---|---|---|---|
| Stripe Payment Links (escolhida) | Zero backend, checkout hospedado, Pix/carto, PCI | Links manual ou via chave restrita; liberação de conteúdo ainda manual | 02/09/2026 |
| Stripe Checkout via API no brisadocs | Entrega automática do conteúdo pago | Chave secreta no servidor, webhooks, mais superfície de manutenção | 02/09/2026 |
| Pix manual (chave + comprovante) | Sem taxa de gateway | Conferência manual, não escala | 02/09/2026 |

Preços definidos pela Adriana em 02/09/2026: R$ 9,90 e R$ 29,90.

## Consequências

- A página está pronta para vender no minuto em que os dois links forem colados
  nas constantes (deploy só da página estática).
- Fluxo de compra: pagamento -> Adriana envia a leitura (momentaneamente
  manual; automação entra como decisão futura se o volume justificar).
- Sem captura de lead, a métrica do funil vira cliques nos botões de pagamento;
  se a Adriana quiser retomar o WhatsApp depois, basta reverter o ADR-0003
  (a API continua íntegra).
