# Boas Práticas Aplicadas — Despensa MVP

Este documento descreve as boas práticas usadas no desenvolvimento da aplicação **Despensa** (gestão de despensa doméstica) e onde cada uma aparece no código. Também registra as limitações conhecidas, para que a avaliação seja honesta.

Arquivo principal: `index.html` (HTML, CSS e JavaScript em um único arquivo).

---

## 1. Arquitetura e organização

| Prática | Como foi aplicada |
|---|---|
| **Arquivo único, sem dependências** | Toda a aplicação está em `index.html`. Não há framework, bundler, `npm install` nem servidor. Basta abrir o arquivo no navegador ou publicá-lo no GitHub Pages. |
| **Encapsulamento do escopo global** | O JavaScript roda dentro de uma IIFE com `'use strict'`, evitando poluir o `window` e variáveis globais acidentais. |
| **Constantes nomeadas** | Chaves de armazenamento (`DB_KEY`, `SESSION_KEY`, `NOTIFIED_KEY`) e a lista de categorias padrão ficam declaradas no topo. |
| **Funções utilitárias pequenas** | Helpers como `$`, `$$`, `money`, `fmtQty`, `daysUntil`, `inviteCode` evitam repetição. |
| **Separação por telas** | Cada aba (Painel, Estoque, Lista de compras, Relatório, Histórico, Grupo) é uma `<section class="view">` controlada por uma única função de roteamento. |
| **Modelo de dados centralizado** | Todo o estado fica em um objeto `db` (`users` e `groups`), e cada grupo guarda itens, histórico, lista de compras e configurações. |
| **Ponto único de gravação** | A função `commit(msg)` persiste os dados, atualiza a tela e mostra o feedback ao usuário. |

## 2. Segurança

| Prática | Como foi aplicada |
|---|---|
| **Escape de HTML na renderização** | A função `esc()` converte `& < > " '` antes de inserir dados do usuário nos templates, reduzindo o risco de XSS. |
| **Senha com hash e salt** | A senha é guardada como SHA-256 de `salt + ':' + senha`, com salt por usuário, e nunca em texto puro. |
| **Fallback documentado** | Se `crypto.subtle` não estiver disponível (contexto não seguro), o código usa FNV-1a. Esse fallback é fraco, e o comentário no código deixa isso explícito. |
| **Confirmação antes de ação destrutiva** | Excluir item, remover membro, sair do grupo, gerar novo código e substituir dados por backup pedem confirmação. |
| **Validação de arquivo importado** | A importação de backup rejeita arquivos que não são JSON válido (mensagem "Arquivo inválido."), e pede confirmação antes de substituir os dados. |
| **Verificação de pertencimento ao grupo** | `currentGroup()` só devolve o grupo se o usuário estiver em `members`, impedindo acesso a grupos alheios pela sessão. |

## 3. Persistência e robustez

| Prática | Como foi aplicada |
|---|---|
| **Tratamento de erros em I/O** | Leitura e escrita do LocalStorage e do sessionStorage estão em `try/catch`. Se o armazenamento estiver cheio ou bloqueado, o usuário recebe um aviso em vez de a página travar. |
| **Validação do dado lido** | `loadDB()` só aceita o dado salvo se tiver as listas `users` e `groups`. Caso contrário, inicia uma base vazia. |
| **Sessão por aba** | A sessão é gravada no sessionStorage, então duas abas podem simular dois membros diferentes. Existe fallback no localStorage. |
| **Sincronização entre abas** | `window.addEventListener('storage', ...)` atualiza a tela quando outra aba altera os dados. |
| **Backup e restauração** | Exportação e importação em JSON na aba Grupo, para migrar ou recuperar dados. |
| **Notificações sem repetição** | `NOTIFIED_KEY` guarda os alertas já notificados, para não disparar a mesma notificação do navegador várias vezes. |
| **Tratamento de falhas de API opcional** | A notificação do navegador e a cópia para a área de transferência estão em `try/catch`, porque podem não existir ou ser negadas. |

## 4. Experiência do usuário (UX)

| Prática | Como foi aplicada |
|---|---|
| **Feedback imediato** | Toda ação mostra uma mensagem curta (`toast`): "Item cadastrado.", "Baixa registrada", "Estoque reposto", "Backup importado." etc. |
| **Mensagens de erro específicas** | Exemplos: "Código não encontrado.", "Arquivo inválido.", "Não foi possível salvar: armazenamento cheio ou bloqueado." |
| **Alertas acionáveis** | O Painel mostra vencidos, vencendo, em falta e estoque baixo, com botões como "Revisar", "Repor" e "Dar baixa". |
| **Lista de compras automática** | Itens em falta ou abaixo do mínimo entram sozinhos na lista, sem cadastro manual. |
| **Busca e filtros combinados** | Estoque pode ser filtrado por texto, categoria e status ao mesmo tempo. |
| **Entrada rápida de dados de teste** | O botão "Entrar com dados de exemplo" permite testar o app sem cadastrar nada. |
| **Rótulos em campos de formulário** | Os campos usam `<label>`. |
| **Responsividade** | Media query em 760px ajusta o layout para telas pequenas. |
| **Tela de convite com código curto** | Código de 6 caracteres sem caracteres ambíguos (sem `0`, `O`, `1`, `I`). |

## 5. Qualidade e manutenção

| Prática | Como foi aplicada |
|---|---|
| **Documentação no README** | `README.md` explica como rodar, funcionalidades, estrutura de dados e limitações. |
| **Comentários nos pontos não óbvios** | Por exemplo, o motivo da sessão em sessionStorage com fallback, e a razão do fallback de hash. |
| **Código sem dependências externas** | Menos risco de quebra por atualização de biblioteca e menos peso na página. |
| **Verificação de sintaxe** | O JavaScript embutido foi checado com `node --check` após a mudança para arquivo único. |
| **Teste manual de fluxos** | Os fluxos principais foram testados no navegador (ver seção 7). |

## 6. Limitações conhecidas

Estas limitações vêm do uso de LocalStorage como banco de dados e da ausência de servidor:

- **Dados só no navegador.** Cada visitante vê apenas os próprios dados. A sincronização entre dispositivos exigiria um backend.
- **Segurança limitada.** Quem acessa o navegador acessa os dados. O hash de senha protege contra leitura simples, mas não substitui autenticação de servidor.
- **Sessão em localStorage como fallback.** Isso mantém o login entre abas, mas também deixa a sessão visível para qualquer script da mesma origem.
- **Fallback FNV-1a.** Não é um hash criptográfico. Só é usado quando o SHA-256 não está disponível.
- **Acessibilidade parcial.** Não foram encontrados atributos ARIA nem navegação completa por teclado nos componentes personalizados. Seria necessário um teste específico com leitor de tela.
- **Sem modo escuro.** O tema é fixo, claro.
- **Sem testes automatizados.** A verificação foi manual, no navegador, e por checagem de sintaxe.

## 7. Verificação realizada

Testes manuais no navegador (Chrome), contra a versão de arquivo único:

- Login com conta de exemplo e criação de conta de teste.
- Cadastro, edição, exclusão e reposição de item.
- Baixa de consumo, com registro no histórico.
- Busca e filtros (texto, status e categoria).
- Lista de compras, relatório mensal e histórico de consumo.
- Criação de grupo, geração de novo código e entrada por convite (com código inválido e válido).
- Remoção de membro pelo responsável e saída do grupo.
- Exportação e importação de backup, incluindo arquivo inválido.
- Troca de conta (Gabriel e Ana) com o mesmo grupo.

Não foram testados: a sincronização em tempo real entre duas abas abertas ao mesmo tempo, notificações do navegador (exigem permissão do usuário) e a interface em telas pequenas.

---

## Conclusão

A aplicação segue práticas básicas e sólidas para um MVP sem servidor: escape de HTML, hash de senha com salt, tratamento de erros de armazenamento, confirmação antes de ações destrutivas, feedback claro ao usuário e distribuição em um único arquivo. As limitações acima são consequência da escolha de LocalStorage e devem ser tratadas na próxima etapa, com um backend.
