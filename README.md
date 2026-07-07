# Arca Digital — Central de Inteligência (Dashboard)

Dashboard de gestão da operação de afiliados (iHub Experts): financeiro, performance por
expert, time comercial e estratégia. Página única (`index.html`), sem build, que lê os dados
ao vivo das planilhas do Google Sheets e tem **login por perfil** via Firebase.

## Arquivos
- **`index.html`** — o dashboard (é isto que é publicado).
- **`COMO-ATIVAR-LOGIN.md`** — passo a passo do login/perfis (Firebase Auth + Firestore).

## Perfis de acesso
- **admin** — vê tudo e gerencia acessos (aba Admin).
- **expert** — vê só a própria performance.
- **comercial** — vê só o Time Comercial.

## Publicar (Cloudflare Pages)
Projeto estático, sem build:
- **Framework preset:** None
- **Build command:** *(vazio)*
- **Build output directory:** `/` (raiz)

O `index.html` na raiz é servido direto. A cada `git push` na branch `main`, o Cloudflare
Pages republica automaticamente.

> Depois de publicar, adicione o domínio (`*.pages.dev`) em
> **Firebase → Authentication → Settings → Authorized domains**.

## Dados
Fonte: 4 planilhas Google (Resultados, Custos de Tráfego, Custos Operacionais, Time Comercial),
lidas ao vivo no navegador. A configuração do Firebase embutida no `index.html` é a config
**web pública** (não é segredo — a segurança vem das regras do Firestore + login).
