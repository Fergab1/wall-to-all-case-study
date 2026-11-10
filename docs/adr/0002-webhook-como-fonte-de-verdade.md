# ADR 0002: O webhook de pagamento é a única fonte de verdade para "pago"

- **Status:** aceita
- **Data:** novembro de 2026

## Contexto

Até a fase de pagamentos, o app decidia sozinho que um pagamento tinha dado certo: um mock de cartão respondia "aprovado", e no PIX o cliente tocava em "Já paguei". Com dinheiro real, isso não é aceitável. O app pode fechar no meio de um checkout, a rede pode cair depois da cobrança e o provedor pode entregar o mesmo evento mais de uma vez.

## Decisão

O webhook assinado da Stripe é o único que escreve o status do pagamento. "Aprovado" na UI significa "a linha do pagamento no banco está como succeeded", lido do banco.

- A função do webhook verifica a assinatura dos dois endpoints (plataforma e contas conectadas) e ignora eventos já processados, usando uma tabela de eventos recebidos.
- Cada evento aplica o status do pagamento e seus lançamentos no ledger em uma única transação. Uma falha responde com erro, para que o provedor reenvie o evento.
- O app consulta o status em um intervalo curto até o pagamento ser concluído ou dar timeout. No PIX, "Já paguei" pergunta ao servidor e avisa quando o pagamento ainda não chegou.
- A notificação ao comerciante sobre um novo agendamento pago é enviada pelo webhook, só na primeira vez que o evento é aplicado, então ela chega mesmo se o cliente tiver fechado o app.

## Alternativas consideradas

- **Confiar no resultado do app.** Rejeitada: um app fechado ou uma resposta perdida deixaria uma cobrança sem agendamento, ou um agendamento marcado como pago sem dinheiro.
- **Assinatura do Realtime em vez de polling.** Mais elegante, mas a camada ao vivo estava fora do escopo desta fase. Polling foi suficiente para o piloto.

## Consequências

- Um evento reenviado não duplica lançamentos no ledger, e um pagamento confirmado com o app fechado continua sendo registrado.
- A UI espera alguns segundos pela confirmação em vez de reagir na hora.
- O schema do banco usa nomes neutros em relação ao provedor, então trocar de provedor em produção não exigiria uma migration destrutiva.
