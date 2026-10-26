# SQPR Gamer Social Work

Hub do jogador e do criador: perfil, conquistas, objetivos, notícias, planner, **amigos**,
**feed** (capturas, chamados para jogar, notícias e promoções), **conexões** (Discord, Steam, Xbox,
PlayStation, Nintendo), **moderação** e configurações de privacidade/conta.

## Estrutura
```
public/        site (o que vai para o ar) — index.html, social.js, feed.js, connections.js, sw.js…
functions/     backend nas Cloudflare Pages Functions (/api/*)
firestore.*    regras e índices do banco
tests/         testes automatizados
.github/       publicação automática no GitHub Pages (só o site, sem backend)
```

## Endereços
- **Cloudflare Pages (completo, com amigos, feed e conexões):** https://sqpr-gamer-social-work.pages.dev
- GitHub Pages (só o hub/planner, sem backend — Amigos, Feed e Conexões precisam do Cloudflare): https://pcmmaster.github.io/SQPR-Gamer-Social-Work/

## Configurar
Tudo o que precisa ser ativado (segredos, regras do Firebase, chaves da Steam/Discord/Xbox)
está em **CONFIGURAR.md** (o feed precisa do bucket R2 `sqpr-posts`, de `OPENAI_API_KEY` e de `ADMIN_UIDS`).

## Atualizar o app
Depois de mudar arquivos em `public/`, aumente o número em `const VERSAO = "sqpr-v11"`
no `public/sw.js` (v11, v12…) para os aparelhos receberem "Nova versão disponível".
