# Acompanhamento Pedidos de Compra

Sistema de gestão de pedidos de compra, RNC (não conformidades) e relatórios
previsto × realizado. Roda como site estático (HTML/CSS/JS puro) com dados
gravados no **Supabase** (Postgres + Autenticação), publicado via **Vercel**
a partir de um repositório **Git**.

## Estrutura dos arquivos

```
gestao-pedidos-compra/
├── index.html          → aplicação completa (HTML + CSS + JS em um único arquivo,
│                          exatamente como nas versões anteriores — só a conexão
│                          com os dados mudou para o Supabase)
├── config.json            → credenciais do Supabase (URL + chave pública) — EDITE ESTE ARQUIVO
├── config.example.json    → modelo de referência do config.json
├── vercel.json             → configuração de deploy na Vercel
├── .gitignore
└── supabase/
    └── schema.sql          → script para criar as tabelas no Supabase
```

O `index.html` é um arquivo único (como antes) para evitar qualquer problema
de caminho quebrado ao abrir o projeto fora de um servidor. Só o `config.json`
fica separado, porque é o arquivo que você edita a cada novo ambiente
(local, homologação, produção).

## Passo 1 — Criar o projeto no Supabase

1. Acesse https://supabase.com, crie uma conta e clique em **New project**.
2. Anote a **senha do banco** que você definir (só é usada internamente pelo Supabase).
3. Depois que o projeto for criado, vá em **SQL Editor → New query**, cole todo o
   conteúdo do arquivo `supabase/schema.sql` deste projeto e clique em **Run**.
   Isso cria as tabelas `pedidos`, `rncs`, `formas_pagamento`, `tipos_frete`,
   `status_pedido`, já com os cadastros padrão e as regras de segurança (RLS).
4. Vá em **Project Settings → API**. Copie:
   - **Project URL** → vai no campo `supabaseUrl`
   - **anon public key** → vai no campo `supabaseAnonKey`
   (Não use a `service_role key` neste arquivo — ela nunca deve ir para o navegador.)

## Passo 2 — Preencher o `config.json`

Abra `config.json` e substitua pelos valores copiados no passo anterior:

```json
{
  "supabaseUrl": "https://SEU-PROJETO.supabase.co",
  "supabaseAnonKey": "sua-anon-key-aqui"
}
```

> A `anon key` é pública por natureza (ela viaja para o navegador de qualquer
> forma); quem protege os dados são as políticas de RLS já criadas pelo
> `schema.sql`, que só liberam acesso a usuários autenticados.

## Passo 3 — Criar os usuários que vão acessar o sistema

O login do sistema usa o **Supabase Auth** (não há mais usuário/senha fixos
no código). Para criar cada pessoa que vai usar o sistema:

1. No painel do Supabase, vá em **Authentication → Users → Add user**.
2. Preencha e-mail e senha e marque **Auto Confirm User** (assim a pessoa já
   consegue entrar sem precisar confirmar e-mail).
3. Repita para cada usuário. Esse é o e-mail/senha que a pessoa vai usar na
   tela de login do sistema.

## Passo 4 — Subir o projeto para o Git

```bash
cd gestao-pedidos-compra
git init
git add .
git commit -m "Primeira versão do sistema"
```

Crie um repositório vazio no GitHub (ou GitLab/Bitbucket) e envie:

```bash
git remote add origin https://github.com/SEU-USUARIO/SEU-REPOSITORIO.git
git branch -M main
git push -u origin main
```

## Passo 5 — Publicar na Vercel

1. Acesse https://vercel.com e faça login com sua conta do GitHub.
2. Clique em **Add New → Project** e selecione o repositório que você criou.
3. Como o projeto é um site estático (sem build), pode deixar as
   configurações padrão da Vercel e clicar em **Deploy**.
4. Em poucos segundos você recebe uma URL pública (algo como
   `https://seu-projeto.vercel.app`) já funcionando com o Supabase configurado.

Sempre que você der `git push` para o branch `main`, a Vercel publica a nova
versão automaticamente.

## Sobre o `config.json` no Git

Por padrão este projeto **comita o `config.json` com as chaves reais**, porque
a `anon key` do Supabase é segura para ficar pública (a proteção de verdade é
o RLS). Se preferir não versionar suas chaves mesmo assim:

1. Adicione `config.json` ao `.gitignore`.
2. Configure o `config.json` diretamente no ambiente de produção — por
   exemplo, subindo o arquivo manualmente pela aba **Deployments → ... →
   Edit Files** da Vercel, ou usando variáveis de ambiente da Vercel com um
   pequeno script de build que gere o `config.json` a partir delas.

## Backup e restauração de dados

Os dados agora vivem no Supabase (não mais no navegador), então não é mais
necessário fazer backup local para não perder informação. Ainda assim, em
**Configurações → Dados (Supabase)** você encontra:

- **Exportar dados (.json)** — baixa uma cópia de tudo (pedidos, RNCs e
  cadastros) para arquivamento.
- **Importar dados (.json)** — lê um arquivo exportado e faz "upsert" no
  banco (atualiza quem já existe pelo `id`, insere o que for novo).

## Rodando localmente antes de publicar

Como o app usa `fetch("config.json")`, ele precisa ser servido por um
servidor HTTP (não funciona abrindo o `index.html` direto com duplo-clique).
Qualquer servidor estático simples resolve, por exemplo:

```bash
npx serve .
# ou
python3 -m http.server 8080
```

Depois acesse `http://localhost:PORTA` no navegador.
