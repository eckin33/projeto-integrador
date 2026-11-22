# Despensa — MVP (Projeto Integrador)

Aplicação web de gestão de despensa doméstica em HTML, CSS e JavaScript puro, com dados salvos no LocalStorage.

## Como rodar
Abra `index.html` no navegador. Não precisa de servidor nem instalação.

Para testar rápido, clique em **"Entrar com dados de exemplo"**:
- demo@despensa.app / 1234 (Gabriel)
- ana@despensa.app / 1234 (Ana, segundo membro do mesmo grupo)

Dica: abra duas abas, entre com um usuário em cada uma e dê baixa em um item. A outra aba atualiza na hora.

## Requisitos funcionais atendidos
1. **Gestão de grupos/famílias:** criar grupo, convidar por código de 6 letras, listar/remover membros, estoque compartilhado.
2. **Cadastro de itens:** produto, quantidade, unidade, categoria, preço pago, validade, estoque mínimo, item essencial. Edição e exclusão.
3. **Baixa de consumo:** qualquer membro dá baixa; registra quem e quando.
4. **Alertas:** vencidos, vencendo (antecedência configurável), em falta, estoque baixo, essenciais. Notificação do navegador opcional.
5. **Relatório mensal:** total gasto, gastos por categoria, compras do mês, comparação com o mês anterior, o que falta repor e estimativa de custo.
6. **Histórico de consumo:** linha do tempo filtrável + padrão de consumo por semana, duração estimada do estoque e sugestão de compra.

Escopo: lista de compras automática (itens em falta, abaixo do mínimo ou vencidos) + itens avulsos, organização por categoria, busca e filtros.

## Funcionalidades e telas

- **Entrar / Criar conta:** login com e-mail e senha. Quem não tem grupo pode criar um ou entrar com o código de convite.
- **Painel:** cards com itens cadastrados, em falta, estoque baixo, vencendo em breve e gasto no mês. Alertas com botões **Revisar**, **Repor** e **Dar baixa**, e a lista de últimas atividades.
- **Estoque:** cards com quantidade, mínimo, validade e último preço. Ações **Dar baixa**, **Repor**, **Editar** e **Excluir**. Busca, filtros por categoria e status, e agrupamento por categoria. **+ Novo item** cadastra um produto.
- **Lista de compras:** sugestões automáticas (em falta, abaixo do mínimo ou vencidos), com o botão **Comprei**. Itens avulsos, com **→ Estoque**, **Remover** e **Copiar lista**.
- **Relatório mensal:** total gasto, compras registradas, baixas de consumo, variação em relação ao mês anterior, gastos por categoria, o que falta repor e as compras do mês.
- **Histórico de consumo:** padrão de consumo por produto, duração aproximada, sugestão de compra e linha do tempo filtrável por tipo de movimentação.
- **Grupo:** código de convite (copiar ou gerar novo), membros (somente o responsável remove), configurações (nome e dias de aviso de validade), categorias, notificações do navegador, exportar/importar backup JSON e sair do grupo.

### Regras dos alertas
Cada item recebe um status. Se mais de um se aplicar, vale o de maior prioridade:
1. **Em falta:** quantidade igual a zero.
2. **Vencido:** validade já passou e há quantidade.
3. **Vence em breve:** validade dentro do prazo de aviso do grupo (padrão 7 dias, de 1 a 60).
4. **Estoque baixo:** quantidade igual ou menor que o mínimo, desde que o mínimo seja maior que zero.

Itens essenciais geram um alerta especial quando acabam.

## Estrutura dos dados (LocalStorage)
- `despensa:db`: `{ users: [...], groups: [{ id, name, code, members, settings, categories, items, logs, shopping }] }`
- `despensa:session`: usuário e grupo ativos (também em sessionStorage, por aba).

## Limitações do MVP (por usar LocalStorage)
- Os dados ficam só no navegador onde foram criados. Membros e "tempo real" funcionam entre abas/janelas do mesmo navegador.
- A senha é guardada como hash SHA-256 com salt, mas sem servidor não existe segurança real: quem acessa o navegador acessa os dados.
- Como os dados já são locais, o app funciona offline. A sincronização entre dispositivos fica para uma etapa com backend.
- Backup: exportar/importar JSON na aba Grupo.
