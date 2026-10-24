# SQPR Gamer Social Work

Hub do jogador e do criador: perfil com foto e capa, perfis de jogo (Steam, PSN,
Xbox…), redes sociais, perseguidor de conquistas, próximos objetivos, jogos que
quer jogar, notícias de jogos e planner pessoal. Com o **modo criador** ligado,
aparecem também Novembro, Vídeos, Roteiro (com teleprompter), Pesquisa,
Checklist e Kit. Funciona offline e pode ser instalado no iPhone, Android e PC.

## Endereço
https://pcmmaster.github.io/SQPR-Gamer-Social-Work/

Repositório: **PCMMASTER/SQPR-Gamer-Social-Work**. Cada envio na branch `main`
publica sozinho pelo GitHub Actions (`.github/workflows/static.yml`).

## Contas
O app abre numa tela de login (Google ou e-mail e senha). Cada pessoa tem o
próprio hub e o próprio planner, salvos na nuvem (Firebase) e privados. Com a
mesma conta, tudo fica igual em todos os aparelhos.

## Instalar
- **iPhone (Safari):** abra o link → Compartilhar → Adicionar à Tela de Início.
- **Android (Chrome):** menu ⋮ → Instalar app.
- **PC (Chrome/Edge):** ícone de instalar na barra de endereço.

## Atualizar o app
1. Suba o `index.html` novo no lugar do antigo.
2. No `sw.js`, aumente o número em `const VERSAO = "sqpr-v5"` (v6, v7…).
3. Ao abrir, o app mostra "Nova versão do app disponível → Atualizar".
