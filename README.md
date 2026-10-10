# Caixa da Casa

Controle financeiro pessoal que roda no navegador (iPhone e Android). Os dados ficam salvos só no aparelho, sem servidor e sem banco de dados.

## Publicar de graça no GitHub Pages

1. Crie um repositório no GitHub (ex.: `caixa-da-casa`). Pode ser privado se sua conta tiver Pages para repositórios privados; senão, público (o código não contém nenhum dado seu, os dados ficam no celular).
2. Envie os arquivos `index.html` e `icon.png` para a raiz do repositório.
3. Vá em **Settings → Pages**, em "Source" escolha **Deploy from a branch**, branch `main`, pasta `/ (root)`, e salve.
4. Em 1 a 2 minutos o site fica em `https://SEU-USUARIO.github.io/caixa-da-casa/`.

## Instalar no celular

- **iPhone (Safari):** abra o link, toque em Compartilhar → "Adicionar à Tela de Início".
- **Android (Chrome):** abra o link, menu ⋮ → "Adicionar à tela inicial" / "Instalar app".

No iPhone, instalar na tela de início é importante: o Safari pode apagar dados de sites que ficam semanas sem ser abertos, e o app instalado não sofre isso.

## Como usar

- **Importar extrato:** no app ou site do banco, exporte o extrato em **OFX** (mais confiável) ou **CSV** e abra pelo botão "Importar extrato". O app categoriza sozinho e ignora lançamentos repetidos.
- **Fatura de cartão:** marque "É fatura de cartão" na tela de revisão se os gastos aparecerem como entrada.
- **Categorias:** ao corrigir a categoria de um lançamento, o app aprende e aplica nos parecidos.
- **Dois celulares:** em Ajustes → "Gerar backup", envie o arquivo pelo WhatsApp e, no outro aparelho, use "Importar backup". Os lançamentos são somados sem apagar nada. Exclusões não passam de um celular para o outro.
- Faça um backup de vez em quando, porque limpar os dados do navegador apaga tudo.
