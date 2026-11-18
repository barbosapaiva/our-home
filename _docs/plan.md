# OurHome — escopo do MVP

**Versão:** 0.2 — decisões de escopo consolidadas  
**Tipo:** app web responsiva, em português, adequada a telemóvel e computador.

## Visão

OurHome é um espaço privado para um agregado familiar coordenar calendário, compras e finanças. A plataforma pode servir vários agregados, mas cada conta pertence a um único agregado.

## Utilizadores e acesso

- **Administrador do agregado:** gere membros, convites/códigos e configurações do agregado.
- **Membros:** colaboram segundo permissões configuradas por membro.
- Um agregado pode adicionar membros por **convite por email ou código**.
- Cada conta pode pertencer a **um único agregado** no MVP.
- Permissões financeiras são configuráveis por membro: quem pode registar, editar e gerir regras financeiras.
- Dados de agregados diferentes ficam isolados; permissões devem ser verificadas no servidor.
- Registos partilhados mostram quem os criou ou alterou.

## Âmbito funcional do MVP

### Painel inicial

Mostrar, num ecrã enxuto:

- próximos eventos;
- pendências de compras e tarefas;
- saldo do mês;
- meses de autonomia financeira.

Os alertas e eventos aparecem quando a pessoa abre a app. **Não há notificações externas** no MVP.

### Calendário familiar

- Criar, consultar, editar e apagar eventos partilhados.
- Eventos recorrentes e um lembrete por evento.
- Eventos próximos aparecem no painel inicial.
- Notificações de email e push ficam fora do MVP; os lembretes são apresentados dentro da app.

### Compras

- Criar várias listas partilhadas.
- Adicionar itens com nome e quantidade.
- Marcar itens como comprados; manter os comprados no histórico da lista.
- Categorias, notas e preços estimados ficam para depois.

### Receitas e despesas

- Registar receitas e despesas manualmente.
- Suportar lançamentos recorrentes simples.
- Lançamentos recorrentes futuros aparecem como **previstos**, entram nas estimativas e precisam de confirmação quando pagos.
- Registos partilhados podem ser associados ao agregado.
- Novos registos de despesas são **partilhados por defeito**; o utilizador pode marcá-los como secretos.
- Permitir resumos mensais e por categoria; preservar histórico de alterações financeiras relevantes.

### Orçamentos

- Definir um limite mensal total por agregado.
- Definir limites opcionais por categoria.
- Mostrar gasto e restante.

### Objetivos

- Suportar objetivos financeiros e familiares.
- Campos essenciais: nome, valor-alvo opcional, prazo e progresso atual.
- Registar contribuições ou atualizações com data e autor, mantendo histórico.
- Após completar a meta de emergência, sugerir uma repartição da poupança pelos objetivos; permitir ajuste manual.

### Resumo financeiro e autonomia

- Resumir receitas, despesas e saldo mensal.
- Calcular meses de autonomia como **saldo disponível para emergência ÷ despesa essencial mensal média**.
- A média usa despesas registadas em categorias marcadas como essenciais; o período de referência é configurável por agregado. Apresentar fórmula, período e data de referência.
- Avisar quando faltarem dados ou o histórico for insuficiente.
- A família define a própria meta de autonomia; não há valor predefinido.
- Investimentos ilíquidos e dinheiro reservado a objetivos não contam para o fundo, salvo inclusão explícita pelo administrador.

### Excedente e regras de alocação

- Calcular o excedente com **receitas menos despesas pagas e compromissos previstos**.
- Permitir substituir o resultado por um valor manual.
- Oferecer sugestões configuráveis com base na regra de 70% para poupança e 5% para alocação identificada pela família como de alto risco.
- Cada agregado escolhe se os 5% fazem parte dos 70% ou se são adicionais.
- Antes de completar a meta do fundo de emergência, as sugestões encaminham a parcela configurada para esse fundo.
- Depois de completar a meta, as sugestões podem distribuir a poupança pelos objetivos, com ajuste manual.
- Mostrar valores planeados e registados. A app **não movimenta dinheiro** nem executa investimentos.
- Guardar histórico das alterações às regras.

### Moedas

- Permitir várias moedas nos registos.
- Cada agregado escolhe uma moeda-base para os totais; a moeda-base pode ser alterada depois de haver lançamentos pagos.
- Converter automaticamente os totais com taxas atualizadas diariamente por um serviço externo. Bloquear registos em moedas para as quais não haja taxa suportada; não aceitar taxas manuais no MVP.
- Preservar a moeda original de cada registo. Lançamentos pagos usam a taxa histórica da data do lançamento, permitindo recalcular valores se a moeda-base mudar.

## Fora do MVP

- Integração bancária, pagamentos, transferências e execução de investimentos.
- Sincronização com calendários externos.
- Projeções financeiras avançadas.
- Importação de ficheiros CSV e leitura de recibos por OCR.
- Notificações por email ou push.
- Categorias, notas e preços estimados nos itens de compras.

## Requisitos de qualidade e confiança

- Interface responsiva, acessível por teclado e com contraste legível.
- Isolamento dos dados por agregado e controlo de permissões no servidor.
- Indicadores financeiros explicáveis: fonte dos dados, fórmula, período e avisos de incompletude.
- Recolher apenas os dados necessários; definir regras para correção, eliminação e exportação de dados.
- Exibir autoria e histórico de alterações relevantes.

## Decisões registadas durante a conversa

1. O MVP cobre coordenação familiar e finanças, de forma simples.
2. O produto deve suportar vários agregados familiares.
3. Entrada no agregado por convite e por código.
4. Novas despesas partilhadas por defeito, com opção de as marcar como secretas.
5. Sugestões configuráveis para a regra de 70% e 5%, sem movimentação de dinheiro.
6. Cada agregado escolhe se os 5% são parte dos 70% ou adicionais.
7. Após a meta de emergência, a app sugere distribuição pelos objetivos, ajustável manualmente.
8. Meta de autonomia definida por cada agregado, sem valor predefinido.
9. Excedente calculado com despesas pagas e compromissos previstos, com opção de valor manual.
10. Várias moedas nos registos, conversão automática diária e moeda-base por agregado.
11. Calendário e listas de compras entram no MVP.
12. Calendário inclui recorrências e um lembrete por evento.
13. Compras incluem várias listas, itens, quantidade e estado de compra.
14. Receitas também são registadas, para calcular saldo e excedente.
15. Orçamentos incluem limite total e limites opcionais por categoria.
16. Objetivos incluem histórico de progresso, data e autor.
17. Autonomia calculada com dados de despesas essenciais registadas, com explicação e avisos.
18. Receitas e despesas podem ser manuais ou recorrentes; recorrências aparecem como previstas e são confirmadas quando pagas.
19. Painel inicial enxuto: próximos eventos, pendências, saldo e autonomia.
20. Permissões financeiras configuráveis por membro.
21. Cada conta pertence a um único agregado no MVP.
22. Sem notificações externas por agora; datas e alertas ficam visíveis dentro da app.

## Decisões adicionais de escopo

### Tarefas

- O painel mostra tanto compras por fazer como tarefas em aberto.
- Incluir uma lista mínima de tarefas com título, responsável e estado de conclusão.

### Despesas pessoais no agregado

- Despesas pessoais podem ser **partilhadas** ou **secretas**; novos registos são partilhados por defeito.
- Despesas partilhadas aparecem no agregado com os respetivos detalhes.
- Despesas secretas entram nos indicadores do agregado como **gastos pessoais não discriminados**. O agregado vê o montante total, mas não vê a descrição nem o detalhe de cada compra. Não é necessário um mínimo de membros para mostrar o total.
- Este modo serve para compras pessoais como roupa ou uma PlayStation: o montante conta para os cálculos familiares, sem expor a compra específica ao outro membro.
- As receitas de cada membro são registadas para permitir os cálculos financeiros do agregado.

### Saldo do fundo de emergência

- A família informa um saldo atual inicial.
- Pode também registar contribuições e levantamentos; os movimentos atualizam o saldo e formam o histórico.

### Conversão cambial

- Saldos e indicadores convertidos usam a taxa diária atual.
- Cada receita ou despesa paga usa a taxa histórica da data do lançamento; os totais podem ser recalculados quando a moeda-base muda. Se a fonte corrigir uma taxa histórica, atualizar o valor armazenado, mesmo que isso altere relatórios anteriores.

### Autonomia, privacidade e moeda-base — decisões adicionais

- Despesas secretas em categorias marcadas como essenciais contam para a média usada no cálculo da autonomia financeira.
- O período usado para a média de despesas essenciais é configurável por agregado, com 6 meses por defeito. Se houver menos histórico, mostrar aviso e o período realmente usado.

- Esclarecimento de privacidade: os gastos pessoais secretos não são ocultados do total do agregado; ocultam-se os detalhes de cada compra.

### Decisões adicionais — moeda, autonomia e gestão

- A moeda-base pode ser alterada mesmo depois de existirem lançamentos pagos. Os valores são recalculados usando a taxa histórica do dia de cada lançamento.
- O período de referência da média de despesas essenciais é configurável por agregado. Ainda falta decidir o valor predefinido inicial.
- Há um administrador de cada vez; o administrador pode transferir o papel para outro membro.
- Cada evento tem no máximo um lembrete no MVP.
- Uma tarefa pode não ter responsável ou ter uma pessoa atribuída.

### Decisões adicionais — histórico cambial

- O período predefinido para a média de despesas essenciais é 6 meses. Se houver menos dados, mostrar um aviso e indicar quantos meses foram efetivamente usados.
- Moedas sem suporte pelo serviço de taxas ficam bloqueadas no MVP; não há alternativa de taxa manual.
- Quando a fonte corrige uma taxa histórica, atualizar o valor armazenado; relatórios históricos podem mudar em consequência.
