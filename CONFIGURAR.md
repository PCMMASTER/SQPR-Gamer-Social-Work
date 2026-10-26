# SQPR Gamer Social Work — perfil público, amizades, conexões e feed

## O que esta entrega faz
- **Perfil público** com `@nome` único (3–20 caracteres), nome de exibição, apresentação,
  jogos favoritos, perfis de jogo que você escolhe mostrar e opção de aparecer ou não na busca.
  E-mail, tokens e dados de login **nunca** vão para o perfil público.
- **Amizades**: buscar por `@nome`, enviar, aceitar, recusar, cancelar, desfazer; listas de
  recebidas/enviadas/amigos em tempo real; se os dois pedirem um ao outro, vira amizade direto.
- **Bloqueio/desbloqueio** (remove amizade e pedidos entre os dois) e **denúncia** de perfil.
- **Verificação de e-mail** exigida para pedir amizade (contas Google já são verificadas).
- **Exclusão de conta** (perfil, @nome, amizades, pedidos, plano na nuvem e login).
- Novas abas: **Amigos**, **Conexões** e **Configurações** (no tema do menu).
- **Conexões** com Discord, Steam, Xbox, PlayStation e Nintendo — botões prontos; cada uma liga
  sozinha quando a chave correspondente for configurada no servidor (veja a seção "Conexões").

- **Feed** (aba nova): Comunidade, Chamados, Notícias, Promoções e Minhas — veja a seção "Feed".
- **Moderação** (aba que só aparece para administradores definidos no servidor).

Ainda NÃO incluído (próximas etapas): reputação, notificações, convite automático para
servidor do Discord, foto de perfil pública, status online/última atividade (não exibidos para ninguém).

## Arquitetura
```
public/            site estático (Cloudflare Pages)  — index.html, social.js, social.css, sw.js…
functions/api/     Cloudflare Pages Functions (backend) — validam tudo e gravam no Firestore
functions/_lib/    código compartilhado (token do Firebase, Firestore REST, limites)
firestore.rules    regras do banco (navegador só LÊ dados sociais; só o servidor grava)
firestore.indexes.json  índice da busca por @nome
tests/             testes automatizados
```
- Login: Firebase Authentication (já existente, projeto `sqpr-hunter`).
- Banco: Cloud Firestore (já existente). Coleções novas: `profiles`, `handles`,
  `friendRequests`, `friendships`, `blocks`, `reports`, `accountStatus`, `rateLimits`.
- O servidor confere a assinatura do token do Firebase em cada chamada, aplica bloqueios,
  impede autoamizade/duplicatas/estados contraditórios e limita frequência
  (30 pedidos/dia, 3 trocas de @/dia, 20 denúncias/dia).

### Endpoints (todos `POST`, com `Authorization: Bearer <ID token do Firebase>`)
| Rota | Corpo | Faz |
|---|---|---|
| `/api/profile/handle` | `{handle}` | cria/troca o @nome (atômico, único) |
| `/api/profile` | `{displayName,bio,favGames[],platforms{},searchable}` | atualiza perfil público |
| `/api/friends/request` | `{to}` | pede amizade (ou aceita se o outro já pediu) |
| `/api/friends/respond` | `{from,accept}` | aceita/recusa pedido recebido |
| `/api/friends/cancel` | `{to}` | cancela pedido enviado |
| `/api/friends/remove` | `{uid}` | desfaz amizade |
| `/api/blocks` | `{uid,block}` | bloqueia/desbloqueia |
| `/api/reports` | `{targetType,targetId,reason,details}` | denúncia |
| `/api/account/delete` | `{confirm:"EXCLUIR"}` | exclui a conta |
| `/api/bootstrap` | `{}` | catálogo, recursos ativos e se é administrador |
| `/api/feed` | `{kind,game,platform}` | feed visível para o usuário |
| `/api/posts/call` | `{game,platform,mode,phrase,slots,level,voice,when,visibility}` | abre chamado |
| `/api/posts/join` | `{id,leave}` | entra/sai de um chamado |
| `/api/posts/shot` | multipart `image`, `caption` | envia captura |
| `/api/posts/manage` | `{id,action:"close"|"delete"|"appeal"}` | só o autor |
| `/api/news` · `/api/deals` | `{apps[]}` · `{}` | notícias da Steam · promoções Steam/Epic |
| `/api/admin/queue` · `/api/admin/decide` | `{}` · `{kind,id,decision,note}` | só administradores |
| `GET /api/img/{id}` | — | imagem aprovada (pública) ou do autor/admin (com login) |

## O que VOCÊ precisa configurar (uma vez)

### 1. Conta de serviço do Firebase (segredo do servidor)
1. Console do Firebase → ⚙ Configurações do projeto → **Contas de serviço** →
   **Gerar nova chave privada** → baixa um `.json`.
2. Guarde esse arquivo em lugar seguro. **Não** coloque no GitHub nem no site.

### 2. Cloudflare Pages
1. Painel Cloudflare → Workers & Pages → seu projeto `sqpr-gamer-social-work` → **Settings**.
2. **Build**: *Build output directory* = `public` (o `wrangler.toml` já indica isso).
   *Build command*: deixe vazio.
3. **Variables and Secrets** (Production) → **Add** → tipo **Secret**:
   - `FIREBASE_SERVICE_ACCOUNT` = cole o conteúdo inteiro do `.json` do passo 1.
4. As variáveis comuns já vêm do `wrangler.toml`:
   - `FIREBASE_PROJECT_ID = "sqpr-hunter"`
   - `ALLOWED_ORIGINS = "https://sqpr-gamer-social-work.pages.dev"` (acrescente seu domínio
     próprio, separado por vírgula, se usar um).
5. **Não** crie `ENVIRONMENT` nem `FIREBASE_AUTH_EMULATOR` em produção (são só para testes).
6. Envie os arquivos (push no GitHub conectado ao Pages). Funções só funcionam com deploy via
   Git ou `npx wrangler pages deploy` — o "arrastar pasta" do painel não publica `functions/`.

### 3. Firebase Authentication
- Authentication → Configurações → **Domínios autorizados** → adicione
  `sqpr-gamer-social-work.pages.dev` (e seu domínio próprio, se houver).
- Modelos de e-mail (opcional): personalize o e-mail de verificação em português.

### 4. Regras e índice do Firestore
Opção A (painel): Firestore → **Regras** → cole o conteúdo de `firestore.rules` → Publicar.
Depois Firestore → **Índices** → Composto → coleção `profiles`, campos
`searchable` (crescente) + `handleLower` (crescente).
E, para o feed, 4 índices na coleção `posts`:
`listed`↑ + `visibility`↑ + `createdAt`↓ · `listed`↑ + `visibility`↑ + `authorUid`↑ + `createdAt`↓ ·
`authorUid`↑ + `createdAt`↓ · `authorUid`↑ + `type`↑ + `status`↑ (a Opção B cria todos sozinha).

Opção B (linha de comando, na pasta do projeto):
```
npm i
npx firebase login
npx firebase deploy --only firestore:rules,firestore:indexes --project sqpr-hunter
```
> A primeira busca por @nome pode mostrar erro por ~1–5 min enquanto o índice é criado.

### 5. Limpeza automática (recomendado)
Firestore → **TTL** → criar política na coleção `rateLimits`, campo `expiresAt`
(apaga contadores antigos sozinho).

## Conexões (Discord, Steam, Xbox, PlayStation, Nintendo)

| Conta | Como conecta | Precisa configurar | O que faz |
|---|---|---|---|
| **Steam** | Login oficial da Steam (OpenID) | **Nada** para conectar. Para importar: `STEAM_API_KEY` | Confirma a conta; importa até 25 jogos mais jogados, horas e % de conquistas |
| **Discord** | OAuth2 oficial, escopo `identify` | `DISCORD_CLIENT_ID`, `DISCORD_CLIENT_SECRET` | Selo “Discord vinculado”; guarda só ID e nome público (token é revogado na hora) |
| **Xbox** | OpenXBL Apps (xbl.io) | `XBL_PUBLIC_KEY`, `XBL_APP_KEY`, `CONNECTIONS_ENC_KEY` | Importa jogos e conquistas; a chave do jogador fica criptografada (AES-GCM) |
| **PlayStation** | Manual (ID online) | — | A Sony não tem API pública para sites; troféus continuam manuais |
| **Nintendo** | Manual (código de amigo) | — | A Nintendo não tem API pública para sites |

Todos os segredos vão em Cloudflare → projeto → **Settings → Variables and Secrets → Add → Secret**.
Depois de adicionar, faça um novo deploy (ou “Retry deployment”) para valer.

### Steam — `STEAM_API_KEY`
1. Entre em https://steamcommunity.com/dev/apikey (conta Steam com compra realizada).
2. Domínio: `sqpr-gamer-social-work.pages.dev` → gere a chave.
3. Cole como segredo `STEAM_API_KEY`.
> O botão **Conectar Steam já funciona sem a chave** (só confirma a conta). Com a chave aparece
> **Importar jogos e conquistas**. O perfil Steam precisa ter “Detalhes de jogos” público.

### Discord — `DISCORD_CLIENT_ID` e `DISCORD_CLIENT_SECRET`
1. https://discord.com/developers/applications → **New Application** (nome: SQPR Gamer Social Work).
2. **OAuth2** → copie **Client ID** e **Client Secret** (Reset Secret).
3. **OAuth2 → Redirects** → adicione exatamente:
   `https://sqpr-gamer-social-work.pages.dev/api/connect/discord/callback`
4. Cole os dois como segredos. Não é preciso bot nem permissões de servidor.

### Xbox — `XBL_PUBLIC_KEY`, `XBL_APP_KEY`, `CONNECTIONS_ENC_KEY`
1. Crie conta em https://xbl.io → **OpenXBL Apps** → crie um app.
2. Redirect/Callback do app: `https://sqpr-gamer-social-work.pages.dev/api/connect/xbox/callback`
3. Copie a **public key** → `XBL_PUBLIC_KEY` e a **app key (privada)** → `XBL_APP_KEY`.
4. Gere a chave de criptografia (32 bytes em base64) e cole em `CONNECTIONS_ENC_KEY`:
   `node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"`
   (guarde-a: se trocar, será preciso reconectar o Xbox).
> O fluxo do OpenXBL foi implementado conforme o padrão “App” deles (auth → claim →
> `X-Authorization` + `X-Contract: 100`). Foi testado contra um simulador, não contra o
> serviço real — confira no primeiro uso; se algo mudou na API deles, é ajuste pequeno em
> `functions/api/connect/[provider]/callback.js` e `sync.js`.

### Segurança das conexões
- Fluxo com `state` aleatório de uso único (10 min), preso ao usuário → protege contra CSRF.
- A identidade é confirmada **no servidor** (Steam `check_authentication`, Discord `/users/@me`,
  OpenXBL `claim`). O navegador nunca vê client secret, chaves ou tokens.
- A mesma conta externa não pode ficar ligada a dois usuários (“conta já vinculada”).
- Desconectar apaga vínculo, token guardado e selo. Excluir a conta apaga todas as conexões.
- Recomendado: Firestore → **TTL** na coleção `oauthStates`, campo `expiresAt`.

## Feed (capturas, chamados para jogar, notícias e promoções)

**Sem texto livre em lugar nenhum.** Quem publica só escolhe opções:
- **Chamado para jogar**: jogo (lista do catálogo), plataforma (só as compatíveis com o jogo),
  modo, frase pronta ("Bora jogar agora?", "Procuro dupla", "Fechando o squad"…), vagas (1–10),
  nível, voz sim/não, agora ou horário marcado (até 7 dias) e quem vê (todo mundo / só amigos).
  O servidor valida tudo e descarta qualquer campo extra (texto, link, anexo). Máximo de 3
  chamados abertos por pessoa, sem duplicados; "agora" vence em 3 h, marcado vence 2 h depois do horário.
  Entrar/sair é controlado no servidor sem corrida (testado: 3 pessoas disputando 1 vaga → só 1 entra).
  Só o autor e quem entrou veem a lista de participantes; os outros veem só a contagem.
- **Captura de jogo**: imagem + legenda pronta opcional. O navegador redimensiona (máx. 1920 px)
  e regrava a imagem, e o **servidor** confere o conteúdo real do arquivo (JPEG/PNG/WebP, 64–4096 px,
  até `MAX_IMAGE_MB`, padrão 5), **remove metadados (EXIF/GPS/XMP/textos)**, guarda em área
  **privada** do R2 e só então manda para a moderação automática.
- **Notícias**: RSS oficial da Steam dos jogos do usuário (links da Steam em Conquistas/fontes do Hub);
  sem jogos, mostra jogos populares. Buscado no servidor, cache de 30 min, sem chave.
- **Promoções**: ofertas da Steam (preços em R$) e jogos grátis da Epic Games. Servidor, cache de 1 h, sem chave.
  As APIs da Steam/Epic usadas são públicas mas **não documentadas oficialmente** — se mudarem, a aba mostra
  "não deu para carregar" sem quebrar o resto.

### Moderação das imagens (servidor)
| Resultado da análise | O que acontece |
|---|---|
| Aprovada | Publicada. Imagem pública em `/api/img/{id}` |
| Imprópria | Recusada. Autor vê o motivo e pode **contestar 1 vez em até 7 dias**; depois disso a imagem é apagada |
| Limítrofe, erro ou **timeout** do serviço | **Fica privada e pendente** para revisão humana (nunca publica sozinha) |
| Suspeita envolvendo menores | **Quarentena**: privada, denúncia automática com prioridade máxima, autor não consegue apagar |

Imagens não aprovadas **nunca** têm URL pública: `/api/img/{id}` devolve 404 para qualquer pessoa
que não seja o autor ou um administrador (conferido no servidor pelo token).
Denúncia de "menor em risco" numa captura tira a imagem do ar na hora e manda para a fila.
Na aba **Moderação**, o administrador aprova ou recusa (recusar exige justificativa); toda decisão
fica em `moderationActions`. Recusar uma quarentena **preserva** a imagem (status `preserved`).

**Procedimento para suspeita de abuso infantil (obrigatório):** não baixe, não compartilhe e não
apague a imagem. Denuncie à SaferNet Brasil (new.safernet.org.br/denuncie) e/ou à Polícia Federal,
guarde o ID da publicação e o registro de moderação, e consulte um advogado sobre a preservação e o
prazo de guarda. Considere ativar também a **CSAM Scanning Tool** da Cloudflare no domínio.

### O que VOCÊ precisa ativar para o feed
1. **R2** (armazenamento privado das capturas): Cloudflare → R2 → *Create bucket* `sqpr-posts`
   (deixe **sem** acesso público). Depois, em `wrangler.toml`, tire o `#` do bloco `[[r2_buckets]]`
   (binding `POSTS_BUCKET`) — ou, no painel do Pages → Settings → Bindings → R2, ligue `POSTS_BUCKET`
   ao bucket `sqpr-posts`. Sem isso, os chamados funcionam e a aba Captura mostra "aguardando ativação".
2. **`OPENAI_API_KEY`** (segredo): chave da OpenAI para a Moderation API (`omni-moderation-latest`,
   gratuita). Sem ela, **toda** captura fica pendente para revisão humana.
3. **`ADMIN_UIDS`** (variável): UIDs do Firebase dos administradores, separados por vírgula
   (Firebase → Authentication → Users → copie o "User UID"). Só esses veem a aba Moderação; a
   permissão é conferida no servidor em toda chamada.
4. Opcional: **`MAX_IMAGE_MB`** (padrão 5).
5. Publique os **índices novos** do Firestore: `firebase deploy --only firestore:indexes`
   (4 índices de `posts`). Sem eles, o Firestore responde com erro pedindo o índice.

## Retenção de dados
- Exclusão de conta apaga: perfil, @nome, amizades, pedidos, bloqueios feitos pela pessoa,
  conexões, publicações e imagens, plano/hub na nuvem e o login.
- Capturas recusadas: a imagem é apagada após o prazo de 7 dias para contestar (ou na hora, se a
  contestação for negada). Imagens em quarentena/preservadas **não** são apagadas automaticamente
  (nem com a exclusão da conta) por possível obrigação legal.
- Mantidos: denúncias (feitas e recebidas), registros de moderação e bloqueios que **outras**
  pessoas fizeram contra a conta — necessários para segurança e eventuais obrigações legais.
  Defina por quanto tempo guardar (sugestão: 12 meses) e documente nos termos de uso.

## Testes
```
npm i
npm run test:api      # 48 verificações das Functions (wrangler + Firestore falso em memória)
node tests/connect.test.mjs     # 30 verificações das conexões (provedores falsos)
node tests/ui-connect.test.mjs  # 14 verificações da tela Conexões no navegador
node tests/auth.test.mjs   # validação de token RS256 (modo produção)
node tests/sa.test.mjs     # assinatura da conta de serviço
node tests/ui.test.mjs     # navegador real: criar @, buscar, pedir, aceitar, perfil, config
npm run test:feed     # 98 verificações do feed (R2 local, moderação falsa, chamados, notícias, promoções)
npm run test:ui-feed  # 28 verificações da aba Feed/Moderação no navegador (desktop e celular)
npm run test:layout   # barra de abas e estouro lateral no celular
npm run test:rules    # regras no emulador OFICIAL do Firestore (precisa de Java)
```
Resultados obtidos ao gerar este pacote: API 48/48, conexões 30/30, token 9/9,
conta de serviço 1/1, interface 16/16, interface de conexões 14/14, feed 98/98,
interface do feed 28/28, barra de abas 15/15, estouro lateral OK. **As regras (`test:rules`) NÃO puderam ser executadas** aqui porque o download
do emulador oficial foi bloqueado — rode `npm run test:rules` antes de publicar as regras.

## Avisos honestos
- A moderação automática foi testada com um **servidor falso** que imita a resposta da OpenAI; a
  chamada real só acontece depois que você colocar `OPENAI_API_KEY`. O mesmo vale para os feeds
  reais da Steam e da Epic (testados com respostas simuladas no formato delas).
- Nenhum filtro automático é perfeito: mantenha alguém de olho na aba Moderação.
- Os testes usaram um Firestore falso em memória e tokens sem assinatura no modo de teste;
  a chamada real ao Google (token da conta de serviço, Firestore de produção e exclusão do
  login no Identity Toolkit) só será exercitada após você configurar o segredo e publicar.
- Faixa etária: por decisão atual, não há idade mínima. Permitir menores exige proteções
  legais adicionais (LGPD art. 14 e legislação sobre menores em ambiente digital) — recomenda-se
  revisão jurídica e deixar claro nos termos de uso.
