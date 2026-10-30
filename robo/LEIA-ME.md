# Mobiliza Pop: site + robô com IA

Portal de alertas de projetos de lei, checagem e monitor do Congresso. Site estático (sem servidor, sem banco de dados), conteúdo em arquivos JSON e um robô que roda sozinho no GitHub, usando o Gemini (chave gratuita do Google) para escrever rascunhos que a equipe aprova. Custo zero.

## Como funciona

```
08h17 e 17h17 (todo dia, no GitHub)
  1. Robô de coleta      → baixa deputados, senadores, votos, projetos novos e notícias
  2. Gemini (escrever-ia.mjs) → atualiza placares, escreve o boletim, sugere alertas
  3. Validador           → barra erro de formato, link quebrado e texto corrompido
  4. Pull request        → a equipe lê, corrige e aprova
  5. Cloudflare Pages    → publica o site sozinho em ~1 minuto
```

Nada vai para o ar sem uma pessoa aprovar.

## O que tem em cada pasta

| Pasta / arquivo | Para quê |
|---|---|
| `site/` | O site. É isso que vai para o ar. |
| `site/data/*.json` | Todo o conteúdo: alertas, checagens, agenda, boletim, parlamentares |
| `robo/coletar.mjs` | Robô de coleta (sem IA) |
| `robo/escrever-ia.mjs` | Escreve o boletim com o Gemini (só aceita links que vieram da coleta) |
| `robo/validar.mjs` | Confere o conteúdo antes de publicar |
| `robo/config.json` | Temas e palavras-chave que o robô monitora. **Edite à vontade.** |
| `CLAUDE.md` | Regras editoriais que qualquer IA segue |
| `AGENTS.md` / `GEMINI.md` | Apontam o Antigravity e o Gemini para as regras |
| `.claude/skills/` | Passo a passo das tarefas: boletim, novo alerta, checagem, revisão de texto |
| `.github/workflows/` | Automação: robô diário e validação |

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

### 2.2 Publicar o site (gratuito)

**Opção A: GitHub Pages** (exige repositório público no plano grátis)
1. **Settings > General**, no fim da página, em **Danger Zone > Change visibility > Public**.
2. **Settings > Pages > Build and deployment > Source: GitHub Actions**.
3. **Actions > Publicar site > Run workflow**. O site sai em `https://SEU-USUARIO.github.io/mobiliza-pop/`.

Depois disso, todo merge na `main` republica o site sozinho, sempre passando pelo validador antes.

**Opção B: Cloudflare Pages** (funciona com repositório privado)
1. Em dash.cloudflare.com: **Workers & Pages > Create > Pages > Connect to Git** e escolha o repositório.
2. Build command: `node robo/validar.mjs`, Build output directory: `site`, variável `NODE_VERSION` = `22`.

## 3. Ligar a IA no GitHub (gratuito)

1. Entre em https://aistudio.google.com/apikey com sua conta Google e clique em **Create API key**. Copie a chave. **Ela é uma senha: não mande para ninguém nem coloque em arquivo do projeto.**
2. No repositório: **Settings > Secrets and variables > Actions > New repository secret**. Name: `GEMINI_API_KEY`, Secret: cole a chave, **Add secret**.

No plano gratuito do Gemini o Google pode usar o que é enviado para melhorar os produtos dele. O robô só envia notícias e dados públicos, nada de dados pessoais.

Para testar no seu PC (PowerShell):
```powershell
node robo/coletar.mjs
$env:GEMINI_API_KEY="cole-a-chave-aqui"; node robo/escrever-ia.mjs
```

### Liberar o robô
Em **Settings > Actions > General > Workflow permissions**:
- marque **Read and write permissions**;
- marque **Allow GitHub Actions to create and approve pull requests**;
- clique em **Save**.

### Testar
**Actions > Robô diário > Run workflow**. Em até 10 minutos aparece o pull request "Rascunhos do robô".

## 4. Rotina da equipe

1. Duas vezes por dia chega um pull request "Rascunhos do robô". O texto do pull request é o relatório da IA: o que mudou, o que conferir e sugestões de novos alertas.
2. Na aba **Files changed**, leia as mudanças.
3. Para corrigir, você pode:
   - editar o arquivo direto no GitHub (ícone de lápis).
4. Tudo certo? **Merge pull request**. O site atualiza sozinho.
5. Não gostou de nada? **Close pull request**. Nada é publicado.

## 5. Usar a IA no seu computador (Antigravity, plano gratuito)

Abra a pasta do projeto no Antigravity e peça ao agente, por exemplo:
> Leia o AGENTS.md e siga as instruções de `.claude/skills/novo-alerta/SKILL.md` para o PL 1234/2026.

| Tarefa | Arquivo de instruções |
|---|---|
| Novo alerta de projeto de lei | `.claude/skills/novo-alerta/SKILL.md` |
| Checagem de notícia | `.claude/skills/nova-checagem/SKILL.md` |
| Revisão de português | `.claude/skills/revisar-texto/SKILL.md` |

Depois envie pelo GitHub Desktop (**Commit** e **Push**). Se o validador apontar erro, peça ao agente para corrigir até passar.

## 6. Antes de lançar

- [ ] Aprovar ou reescrever o manifesto (em `site/index.html`) e trocar `manifesto_aprovado` para `true` em `site/data/config.json`
- [ ] Preencher `canal_whatsapp` e `email_contato` em `site/data/config.json`
- [ ] Conferir no texto oficial o capítulo "Por que preocupa" do PL 3615/2019 (está marcado como provisório)
- [ ] Revisar os temas e palavras-chave em `robo/config.json`
- [ ] Apagar os modelos de checagem quando a primeira checagem real for publicada (a skill `/nova-checagem` faz isso)
