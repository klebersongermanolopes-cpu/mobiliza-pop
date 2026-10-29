# Mobiliza Pop: site + robô com IA

Portal de alertas de projetos de lei, checagem e monitor do Congresso. Site estático (sem servidor, sem banco de dados), conteúdo em arquivos JSON e um robô que roda sozinho no GitHub, usando o Claude Code para escrever rascunhos que a equipe aprova.

## Como funciona

```
08h17 e 17h17 (todo dia, no GitHub)
  1. Robô de coleta      → baixa deputados, senadores, votos, projetos novos e notícias
  2. Claude (/boletim-diario) → atualiza placares, escreve o boletim, sugere alertas
  3. Validador           → barra erro de formato, link quebrado e texto corrompido
  4. Pull request        → a equipe lê, corrige (ou pede ao @claude) e aprova
  5. Cloudflare Pages    → publica o site sozinho em ~1 minuto
```

Nada vai para o ar sem uma pessoa aprovar.

## O que tem em cada pasta

| Pasta / arquivo | Para quê |
|---|---|
| `site/` | O site. É isso que vai para o ar. |
| `site/data/*.json` | Todo o conteúdo: alertas, checagens, agenda, boletim, parlamentares |
| `robo/coletar.mjs` | Robô de coleta (sem IA) |
| `robo/validar.mjs` | Confere o conteúdo antes de publicar |
| `robo/config.json` | Temas e palavras-chave que o robô monitora. **Edite à vontade.** |
| `CLAUDE.md` | Regras editoriais que a IA segue sempre |
| `.claude/skills/` | Tarefas prontas da IA: `/boletim-diario`, `/novo-alerta`, `/nova-checagem`, `/revisar-texto` |
| `.github/workflows/` | Automação: robô diário, `@claude` nos comentários e validação |

## 1. Ver o site no seu computador

Precisa do [Node.js](https://nodejs.org) versão 22 ou mais nova.

```bash
npm run site
```

Abra http://localhost:8080. Para testar o robô de coleta: `npm run coletar`. Para conferir o conteúdo: `npm run validar`.

## 2. Colocar no ar

### 2.1 GitHub
1. Crie um repositório **privado** no GitHub, por exemplo `mobiliza-pop`.
2. Na pasta do projeto:
   ```bash
   git init
   git add .
   git commit -m "Primeira versão do Mobiliza Pop"
   git branch -M main
   git remote add origin https://github.com/SEU-USUARIO/mobiliza-pop.git
   git push -u origin main
   ```
   Ou peça ao Claude Code: "suba este projeto para um repositório novo no GitHub".

### 2.2 Cloudflare Pages (gratuito)
1. Em dash.cloudflare.com: **Workers & Pages > Create > Pages > Connect to Git** e escolha o repositório.
2. Configuração do build:
   - Build command: `node robo/validar.mjs`
   - Build output directory: `site`
   - Variável de ambiente: `NODE_VERSION` = `22`
3. Salve. O site sai num endereço `*.pages.dev`. Em **Custom domains** você liga o domínio (ex.: mobilizapop.com.br).

Como o build roda o validador, conteúdo com erro nunca chega ao ar.

## 3. Ligar a IA no GitHub

O robô usa a **assinatura do Claude** (Pro ou Max) de quem configurar. Não precisa de chave de API paga à parte.

1. Abra o terminal na pasta do projeto (pode ser o terminal do Antigravity) e rode `claude`.
2. Digite `/install-github-app` e siga as instruções:
   - instale o app do Claude no repositório;
   - quando perguntar a autenticação, escolha **criar um token com a sua assinatura**. Ele salva o segredo `CLAUDE_CODE_OAUTH_TOKEN` no repositório;
   - se oferecer arquivos de workflow, pode pular: eles já estão no projeto.
3. No GitHub, vá em **Actions > Robô diário > Run workflow** para testar na hora.
4. Em poucos minutos aparece um pull request "Rascunhos do robô".

Se preferir fazer à mão: rode `claude setup-token` no seu PC, copie o token e crie o segredo `CLAUDE_CODE_OAUTH_TOKEN` em **Settings > Secrets and variables > Actions**. Instale também o app em https://github.com/apps/claude.

**Importante:** o uso do robô entra na cota da assinatura de quem gerou o token. Se o volume crescer, dá para trocar por uma chave de API (instruções dentro de `.github/workflows/robo-diario.yml`).

## 4. Rotina da equipe

1. Duas vezes por dia chega um pull request "Rascunhos do robô". O texto do pull request é o relatório da IA: o que mudou, o que conferir e sugestões de novos alertas.
2. Na aba **Files changed**, leia as mudanças.
3. Para corrigir, você pode:
   - editar o arquivo direto no GitHub (ícone de lápis), ou
   - comentar pedindo ao Claude: `@claude troque o título do segundo item por algo mais curto`.
4. Tudo certo? **Merge pull request**. O site atualiza sozinho.
5. Não gostou de nada? **Close pull request**. Nada é publicado.

## 5. Usar a IA no seu computador

Com o Claude Code aberto na pasta do projeto:

| Comando | O que faz |
|---|---|
| `/novo-alerta PL 1234/2026` | Cria a página de alerta a partir das fontes oficiais |
| `/nova-checagem "frase que está circulando"` | Investiga e escreve a checagem com veredito e fontes |
| `/revisar-texto` | Revisa o português de todo o site sem mudar fatos |
| `/boletim-diario` | Escreve o boletim (rode `npm run coletar` antes) |

Depois peça: "crie um pull request com essas mudanças". A revisão segue igual.

Esses comandos também funcionam no GitHub: abra uma issue escrevendo `@claude /novo-alerta PL 1234/2026`.

## 6. Antes de lançar

- [ ] Aprovar ou reescrever o manifesto (em `site/index.html`) e trocar `manifesto_aprovado` para `true` em `site/data/config.json`
- [ ] Preencher `canal_whatsapp` e `email_contato` em `site/data/config.json`
- [ ] Conferir no texto oficial o capítulo "Por que preocupa" do PL 3615/2019 (está marcado como provisório)
- [ ] Revisar os temas e palavras-chave em `robo/config.json`
- [ ] Apagar os modelos de checagem quando a primeira checagem real for publicada (a skill `/nova-checagem` faz isso)
