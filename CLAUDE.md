# Caixa da Casa

App pessoal de controle financeiro (inspirado no Pierry) para uso do casal: iPhone (Safari) e Android (Chrome). Interface toda em português do Brasil.

## Restrições do projeto
- **Sem backend e sem banco de dados.** Tudo roda no navegador; os dados ficam no `localStorage` do aparelho (chave `caixa-da-casa-v1`).
- **Sem build.** Um único `index.html` com CSS e JS inline. Única dependência externa: Google Fonts (Familjen Grotesk + Figtree), com fallback de sistema.
- Hospedagem: GitHub Pages (estático). `icon.png` é o ícone da tela de início do iPhone.
- Tem que funcionar bem em tela de celular (~375–430px), respeitar safe-area (notch) e ter modo claro/escuro via tokens CSS em `:root`.
- Não há conexão automática com bancos (Open Finance exige empresa autorizada + servidor). Os dados entram por importação de extrato OFX/CSV ou lançamento manual.

## Estrutura do código (index.html)
- `CATS`: categorias com cor, tipo (`out`/`in`) e palavras-chave para categorização automática (marcas comuns de Salvador/BA incluídas).
- Estado `S`: `{accounts, tx, budgets, bills, goals, rules, demo, updatedAt}`.
  - `tx`: `{id, date:"AAAA-MM-DD", desc, amount (negativo = gasto), acc, cat, fitid?, demo?}`
  - `rules`: mapa descrição normalizada → categoria (aprendido quando o usuário corrige uma categoria).
  - `bills[].paid[AAAA-MM]`: `true` ou id da transação lançada ao marcar como paga.
- Primeira abertura carrega dados de exemplo (`seedDemo`, `S.demo=true`); "Começar do zero" limpa.
- Telas (`vHome`, `vTx`, `vBudget`, `vBills`, `vGoals`) geram HTML por string; `render()` redesenha tudo. Formulários abrem em "sheet" inferior (`openSheet`).
- Ações por delegação de clique em `data-act`.
- Importação: `parseOFX`, `parseCSV` (detecta separador, cabeçalho, formato BR de número/data, fatura de cartão estilo Nubank `date,title,amount`), deduplicação por `dupKey`/FITID.
- Sincronização entre os dois celulares: backup JSON exportado (Web Share / download / copiar texto) e importado com **merge** (`mergeBackup`), sem apagar dados locais.
- `window.__caixa` expõe os parsers para teste.

## Convenções
- Valores em BRL com `Intl.NumberFormat("pt-BR")`; aceitar entrada "1.234,56".
- Toda cor vem dos tokens CSS; nada de cor fixa que só funcione em um tema.
- Envolver acesso ao `localStorage` em try/catch.
- Não usar `alert/confirm/prompt`; confirmações são feitas com `askConfirm` dentro da página.
