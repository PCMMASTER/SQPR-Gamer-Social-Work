# SQPR Hunter — app (PWA)

Planner completo: Hoje, Tarefas, Rotina, Semana, Casa, Novembro, Vídeos,
Roteiro (com teleprompter), Pesquisa, Checklist e Kit. Funciona offline e
pode ser instalado no iPhone, Android e computador.

## Endereço do app
https://pcmmaster.github.io/sqpr/

Repositório: **PCMMASTER/sqpr** (GitHub Pages, branch `main`, pasta raiz).

## Login e sincronização
O app abre numa tela de login (Google ou e-mail e senha). Com a mesma conta,
o plano fica igual em todos os aparelhos (Firebase, projeto `sqpr-hunter`).

## Instalar
- **iPhone (Safari):** abra o link → Compartilhar → Adicionar à Tela de Início.
  Use sempre pelo ícone: ele guarda os dados separado do Safari.
- **Android (Chrome):** menu ⋮ → Instalar app / Adicionar à tela inicial.
- **PC (Chrome/Edge):** ícone de instalar na barra de endereço, ou menu → Instalar.

## Dados
O plano fica salvo no aparelho e na nuvem da sua conta. Sem internet, continua
salvando no aparelho e envia quando a conexão voltar. **Kit → Baixar backup**
gera uma cópia extra em arquivo.

## Atualizar o app
1. Suba o `index.html` novo no lugar do antigo.
2. No `sw.js`, aumente o número em `const VERSAO = "sqpr-v3"` (v4, v5…).
3. Ao abrir, o app mostra "Nova versão do app disponível → Atualizar".
   Os dados salvos continuam.
