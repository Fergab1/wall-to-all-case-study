# ADR 0004: A plataforma retém os fundos e o saque vira transferências

- **Status:** aceita
- **Data:** novembro de 2026

## Contexto

O design de pagamentos usava o Stripe Connect. O modelo-alvo era destination charge: cada pagamento de agendamento enviaria na hora a parte do comerciante para a conta conectada dele, e "sacar" dispararia sob demanda um repasse dessa conta para o banco do comerciante. Não estava confirmado como isso funciona para uma plataforma brasileira, principalmente com PIX, então a primeira tarefa do plano foi um teste no sandbox antes de escrever código de servidor.

O teste confirmou cobranças por cartão com tarifa da plataforma, cobranças PIX e o onboarding de novas contas conectadas. Também mostrou dois bloqueios: repasses sob demanda (manuais) não são permitidos para contas conectadas brasileiras, que recebem repasses automáticos no calendário do provedor, e toda transferência da plataforma precisa apontar para uma cobrança de origem.

## Decisão

A plataforma retém os fundos. Toda cobrança online (agendamento por cartão ou PIX, avanço na fila) é feita na conta da plataforma, e o ledger credita o comerciante quando o pagamento é bem-sucedido.

- "Sacar", nas finanças do comerciante, vira uma ou mais transferências da plataforma para a conta conectada do comerciante. Cada transferência usa como origem uma cobrança desse comerciante que ainda não foi repassada, da mais antiga para a mais nova, até cobrir o valor líquido da cotação.
- A tarifa de saque simplesmente não é transferida. Nenhum estorno é necessário.
- O repasse para o banco continua automático. A tela de saque mantém o design, e só o prazo de entrega prometido mudou.
- O saque é protegido por uma chave de idempotência vinda do app. Uma falha antes da primeira transferência reverte o lançamento no ledger, e uma falha no meio mantém o saque em processamento, para que uma nova tentativa com a mesma chave o retome.
- Saques de cashback de indicação feitos por comerciantes usam o mesmo mecanismo. Saques de cashback de clientes são registrados para tratamento manual no piloto.

## Alternativas consideradas

- **Destination charges com repasses sob demanda.** O design original. Não é possível no Brasil, como o teste mostrou.
- **Destination charges só com repasses automáticos.** Tiraria o "sacar" do produto e entraria em conflito com as telas de finanças do comerciante aprovadas pelo cliente.

## Consequências

- O ledger e o provedor continuam conciliáveis: cada transferência está ligada a uma cobrança específica.
- Os saques precisam de uma etapa de alocação e de um lock, e a máquina de estados deles tem mais casos para testar.
- A decisão depende das regras do provedor para o Brasil. Se elas mudarem, o modelo de fundos é a primeira coisa a revisitar.
