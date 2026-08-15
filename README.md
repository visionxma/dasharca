# Arca Digital — Central de Inteligência (Dashboard)

Dashboard de gestão da operação de afiliados (iHub Experts): financeiro, performance por
expert, time comercial e estratégia. Página única (`index.html`), sem build, que lê os dados
ao vivo das planilhas do Google Sheets e tem **login por perfil** via Firebase.

## Arquivos
- **`index.html`** — o dashboard (é isto que é publicado).
- **`COMO-ATIVAR-LOGIN.md`** — passo a passo do login/perfis (Firebase Auth + Firestore).
- **`FUNIL-EXPERTS.md`** — o board de onboarding de experts: regras do Firestore, etapas,
  prazos e o tratamento das credenciais sensíveis.

## Perfis de acesso
- **admin** — vê tudo, gerencia acessos (aba Admin) e é o único que revela credenciais.
- **expert** — vê só a própria performance.
- **comercial** — vê só o Time Comercial.
- **equipe** — vê só o Funil de Experts (Camily, William, Márcio).

## Funil de Experts
Aba com board de 6 etapas (Entrada → Organização → Alinhamento → LP → Publicação → Rodando).
Cada expert é um card com **um responsável único**, que troca sozinho quando a etapa muda, e
com selo vermelho quando passa do prazo. Diferente do resto do dashboard, estes dados são
**gravados no Firestore**, não lidos das planilhas. Ver `FUNIL-EXPERTS.md`.

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
