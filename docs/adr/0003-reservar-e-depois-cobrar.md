# ADR 0003: Reservar o horário primeiro e cobrar depois

- **Status:** aceita (substitui a ordem "cobrar primeiro, gravar depois" da fase de agendamento)
- **Data:** novembro de 2026

## Contexto

Na fase de agendamento (outubro de 2026), a confirmação seguia "cobrar primeiro, gravar depois": pagamento, depois criação do agendamento, depois registro do pagamento. Um cartão recusado nunca criava agendamento. Mas, se a cobrança desse certo e o insert do agendamento falhasse, o cliente seria cobrado sem agendamento e sem reembolso. Com os pagamentos ainda simulados, essa ordem foi mantida de propósito, e a inversão ficou planejada para a fase de pagamentos.

## Decisão

Inverter a ordem quando os pagamentos se tornaram reais:

1. Criar o agendamento como pendente, sem notificar o comerciante. Se isso falhar, nada foi cobrado.
2. Cobrar com uma referência a esse agendamento e uma chave de idempotência derivada dele.
3. Um cartão recusado reaproveita a mesma reserva e a mesma chave, então tentar de novo com outro cartão ou com PIX nunca duplica o horário nem a cobrança.
4. Se o cliente desistir (voltar, fechar o sheet do PIX ou mudar para pagamento no local), a reserva não paga é cancelada.
5. Se o app morrer no meio, o webhook ainda registra o pagamento, e uma rotina do banco libera as reservas que continuam sem pagamento depois do limite de tempo.

## Alternativas consideradas

- **Continuar cobrando primeiro.** Rejeitada: deixa uma cobrança paga sem agendamento sempre que a escrita falha.
- **Criar o agendamento dentro do webhook depois do pagamento.** Rejeitada: duplicaria no servidor a validação do agendamento e as checagens de conflito, e exigiria reembolso sempre que o horário fosse ocupado durante um pagamento PIX lento.

## Consequências

- Um cliente nunca é cobrado sem agendamento, nem cobrado duas vezes pelo mesmo agendamento.
- Por um curto período, o comerciante pode ver um agendamento pendente que ainda não foi pago.
- A lógica de agendamento tinha sido movida para um módulo próprio e coberta por um teste de caracterização na fase de agendamento, o que deixou essa mudança segura de fazer.
