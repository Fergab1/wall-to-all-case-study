# ADR 0001: Construir o novo escopo com UI primeiro, atrás de interfaces de serviço

- **Status:** aceita
- **Data:** julho de 2026

## Contexto

Em julho de 2026 o cliente aprovou um escopo bem maior: fila presencial ao vivo com opção paga para avançar, carteira do cliente e finanças do comerciante, programa de indicação com cashback, feed de fotos e vídeos e assinatura para o comerciante. Isso significava cerca de 60 telas e estados novos nos dois lados do app. O app existente não tinha nenhuma das capacidades necessárias: nem provedor de pagamento, nem mapas, nem câmera ou vídeo, nem fila ao vivo. O próprio provedor de pagamento ainda não estava decidido.

## Decisão

Entregar cada fase com UI primeiro, com os dois lados de uma funcionalidade na mesma fase, e colocar cada domínio atrás de uma interface de serviço (fila, carteira, indicação, feed, assinatura, pagamentos de agendamento).

- As telas consomem só a interface, por meio de um provider na raiz do app. Dados fixos dentro de uma tela não são permitidos.
- Cada interface ganha uma implementação em memória com dados fixos e timers que simulam atualizações ao vivo (a fila andando, um ticket sendo chamado). Os testes sempre usam essa implementação.
- As implementações reais entram depois, ao lado dos mocks, e o provider da raiz escolhe uma por configuração. Trocar uma pela outra não mexe em nenhuma tela.
- As bibliotecas nativas (mapas, câmera, vídeo) foram instaladas já na primeira fase, para que o development client fosse recompilado uma única vez.

## Alternativas consideradas

- **Construir cada funcionalidade junto com o backend.** Rejeitada: toda tela de pagamento teria esperado a decisão do provedor, e o cliente ficaria semanas sem ver nada.
- **Telas estáticas sem interface.** Rápido no início, mas toda tela precisaria de retrabalho quando os dados reais chegassem.

## Consequências

- As fases 0 a 6 foram entregues entre julho e o início de novembro de 2026, antes de existir qualquer pagamento real, e o cliente pôde revisar fluxos completos cedo.
- Quando os pagamentos chegaram, o código do provedor ficou só nas funções de servidor e em uma pasta do app. O resto do app manteve seus contratos, e os testes de tela continuaram rodando nas implementações em memória.
- Cada domínio tem duas implementações para manter alinhadas. Todos os domínios já rodam na implementação real, e a implementação em memória continua sendo usada pelos testes.
