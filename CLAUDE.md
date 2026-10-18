# Caixa da Casa

App pessoal de controle financeiro (inspirado no Pierry, mais simples) para uso do casal: iPhone (Safari) e Android (Chrome). Interface toda em português do Brasil, linguagem simples. A dona é MEI prestadora de serviços.

## Restrições do projeto
- **Sem backend e sem banco de dados.** Tudo roda no navegador; dados no `localStorage` (chave `caixa-da-casa-v1`, `S.v = 2`; `migrate()` atualiza dados antigos).
- **Sem build.** Um único `index.html` com CSS e JS inline. Dependências externas: Google Fonts e, carregados sob demanda: pdf.js 3.11.174 e SheetJS 0.18.5 (cdnjs) e Tesseract.js 5.1.1 (jsdelivr, leitura de prints; baixa o idioma português na primeira vez e só funciona no site publicado, não na prévia do Claude).
- Hospedagem: GitHub Pages. `icon.png` é o ícone da tela de início.
- Visual: rosa bebê e fofo (Fredoka nos títulos, Nunito no texto, cantos bem arredondados, botões em pílula, porquinho na tela de entrada). Pouca informação por tela. Cores só por tokens CSS em `:root` (com modo escuro).
- Dependências de fonte: Google Fonts (Fredoka + Nunito).
- Não há conexão automática com bancos (Open Finance exige empresa autorizada). Dados entram por arquivo (PDF/OFX/CSV/XLSX), texto colado ou "escrever rápido".

## Regras fiscais (objeto `RULES`, no topo do script)
- Teto MEI R$ 81.000/ano (proporcional no ano de abertura: teto/12 × meses). Até 20% acima: DAS complementar e vira ME em janeiro. Acima de 20%: desenquadramento retroativo a janeiro. PLP 108/2021 (R$ 130 mil) ainda não aprovado; o teto é editável em Configurar MEI.
- DAS MEI serviços 2026: R$ 86,05 (5% do salário mínimo de R$ 1.621 + R$ 5 ISS), vence dia 20.
- IR: 32% da receita de serviços é isenta; tributável = (receita − despesas da conta PJ) − parte isenta, + salário. Obrigação de declarar acima de R$ 35.584 (declaração 2026). Imposto estimado com desconto simplificado (20%, limite R$ 17.640), tabela anual 2026 e redutor da Lei 15.270/2025 (zero até R$ 60 mil; redução 8.429,73 − 0,095575 × renda até R$ 88.200).

## Estrutura do código
- `CATS` categorias com palavras-chave; `propria` = transferência entre contas próprias (fica fora dos totais); `mei` = ganho do MEI.
- `BANKS` lista de bancos com palavras-chave (detecção de banco no cabeçalho do arquivo e no PDF do Registrato).
- Estado `S`: `{profile:{name}, deviceId, people:[{id,name}], accounts:[{id,name,type:'pf'|'pj',bank,owner}], tx, budgets, bills:[{...,owner:'me'|personId|'shared'}], goals, rules, mei:{on,declined,open,teto,clients,notClients,das,salaryManual}, benefits:{on,card,since,items:[{id,name,amount,day,mei}],machine,accId,skip}, regBanks, onb, showWelcome, look:{palette,mode:'auto'|'light'|'dark'}, pet:{animal,bodyColor,spots,top,topColor,pants,pantsColor,shoes,shoesColor,hair,hairColor}, demo}`.
- `meiClass(t)`: `mei` | `no` | `maybe`. Conta PJ → tudo é MEI (exceto rendimento/estorno/transferência própria). Conta pessoal → pergunta uma vez por pagador (`payerKey`) e grava em `mei.clients`/`notClients`.
- `isOwnText` reconhece transferência própria pelo nome completo da usuária e palavras como "mesma titularidade".
- `meiYear(ano)` e `irCalc(m)` fazem os cálculos; `vMei()` desenha a aba.
- Importação: `parseOFX`, `parseCSV`, `parseLoose` (texto livre de PDF/colado/print), `parseQuick` ("uber 23,50"). `handleFiles` aceita vários arquivos de uma vez, inclusive imagens; PDF sem texto (print salvo em PDF) e imagens passam por `prepCanvas` (cinza + contraste) e `ocrCanvases`.
- `parseLoose`: datas em linha própria valem para as linhas seguintes; `SKIP_RE` ignora linhas de resumo de fatura (limite, parcelas, utilizado, juros de parcelamento…); `STOP_RE` corta seções de rodapé; `NODATE_RE` ignora datas de vencimento/período/emissão; valores com `%` não contam; aceita "(+)/(-)" do Banco do Brasil; quando a descrição vem antes ou depois do valor (Caixa, BB), junta as linhas. Em modo `ocr` o sinal é decidido pelo texto (só "crédito recebido", "estorno" etc. são entrada). Em fatura de cartão, valores sem sinal já são gastos (`guess`), e os com sinal são invertidos.
- Testado com extratos reais (Inter PF de 26 páginas: soma bate com o saldo final), faturas Nubank e Porto Seguro, extrato BB, comprovante da Caixa e print do Swile. Tudo passa pela tela "Conferir" (`impSheet`), com deduplicação (`dupKey`/FITID) e marcação automática de contas fixas pagas (`autoMatchBills`).
- Telas: `vHome`, `vTx`, `vMei`, `vPlan` (Limites/Contas/Metas). Formulários em folhas (`openSheet`). Ações por `data-act`.
- `window.__caixa` expõe parsers e cálculos para teste.

## Tela de entrada e primeiros passos
- `#screen` é uma tela cheia por cima do app. `openWelcome()` mostra a entrada com dois botões: Primeiros passos e Ir para o início. Abre ao iniciar enquanto `S.showWelcome` for true (desliga ao terminar o assistente; a própria tela tem a opção). O coração no topo e Ajustes reabrem.
- Assistente `WSTEPS = intro, voce, mei, bancos, extrato, contas, uso`; cada etapa em `WSTEP[key]()` devolve título, texto e corpo; `wzSave(key)` grava; "Pular" avança sem gravar (só intro e uso não têm pular). Progresso em `S.onb = {done, step}`.
- Na etapa de extrato, a importação usa o mesmo fluxo do botão + (`ui.forceAcc` pré-seleciona o banco) e volta para o assistente.

## Família
- Cada aparelho é dono dos seus dados (`owner: "me"`). Ao importar o backup de outra pessoa (`mergeBackup`), as contas dela ganham `owner` = id em `S.people` (criado pelo nome do perfil do backup; se faltar nome, o app pergunta). Mesmo `deviceId` ou mesmo nome = a própria pessoa (outro aparelho).
- `ownerOf(t)` = `t.owner` ou dono da conta. Pix que cita o nome completo de outro membro vira categoria `familia`: conta para a pessoa, mas fica fora do total da família (`counts`).
- `ui.who` = `all` | `me` | id da pessoa; `whoBar()` troca a visão; `txOf` filtra por `inScope`. Contas fixas podem ser de uma pessoa ou `shared` (metade para cada, `billShare`).
- MEI e Imposto de Renda usam só os lançamentos de `me`.
- Quem respondeu "Não sou MEI" (`mei.declined`) não vê a aba MEI; no lugar aparece Ajustes.

## Benefícios da empresa
- Cartão de benefícios (ex.: Swile) não tem extrato. Em `benefitsSheet()` a usuária cadastra os valores fixos (nome, valor, dia, se conta no MEI) e o mês de início; `syncBenefits()` cria um lançamento por mês na conta `kind:"beneficio"` (chave `benefitKey`; excluídos ficam em `benefits.skip`). Roda ao abrir o app.
- Maquininha (`MACHINE_RE`, ex.: "Credito domicilio cartao: ANTECIPACAO - INTER PAG"): `mei.machine` (padrão ligado) faz as vendas contarem no MEI em qualquer conta; `mei.machineOwn` ("são do meu próprio cartão de benefícios") as marca como `propria`, para não somar de novo na família. Cartão de benefícios (conta `kind:"beneficio"`, ex.: Swile) não conta no MEI.

## Aparência
- Paletas por `data-palette` no `<html>` (lista `PALETTES`): rosa (padrão), bege, marrom, terracota, dourado, musgo e douradoBranco têm versão clara e escura, cada uma feita de vários tons da mesma cor; `preto` (preto e dourado) e `branco` são fixas (sempre escura/clara). Os fundos de aviso (`--ok-soft`, `--warn-soft`, `--bad-soft`) são misturados com `color-mix` a partir da cor da superfície, para combinar com qualquer paleta. `S.look.mode` força claro/escuro via `data-theme`. `applyLook()` aplica.
- Mascote `petSVG(p)`: porquinho, patinho, vaquinha, gatinho peludo (pinta diagonal em metade do rosto, `fur()` desenha o pelo) ou cachorrinho (com ou sem pintinhas, `pet.spots`); `pet.bodyColor` troca a cor do bicho (vazio = cor original); com camisa, vestido decote V, vestido com babado, calça, tênis, salto e perucas (liso curto/longo, cacheado curto/longo), cada peça com cor. Editado em `petSheet()`; aparece na tela de entrada e dentro do círculo dos cartões "Sobrou em…" e "Falta pagar em…" (classe `hero pet`).

## Convenções
- Valores em BRL com `Intl.NumberFormat("pt-BR")`; aceitar "1.234,56".
- Envolver `localStorage` em try/catch. Não usar `alert/confirm/prompt` (usar `askConfirm`).
- Textos curtos e sem jargão; o que é estimativa deve dizer que é estimativa.
