# Como ativar o Login (Firebase) — Arca Digital Dashboard

O dashboard já funciona **sem login** (modo público). Para ligar o **login com perfis
de acesso** (admin / expert / comercial), siga este passo a passo. É **gratuito** (plano
Spark do Firebase). Leva ~15 minutos e só precisa ser feito **uma vez**.

> Dica: faça tudo no computador, logado na conta Google que vai ser a "dona" do sistema.

---

## Passo 1 — Criar o projeto no Firebase
1. Acesse **https://console.firebase.google.com** e clique em **Adicionar projeto**.
2. Nome: `arca-dashboard` (ou o que preferir). Pode desativar o Google Analytics.
3. Clique em **Criar projeto**.

## Passo 2 — Ativar o login por E-mail/Senha
1. No menu esquerdo: **Criação → Authentication → Vamos começar**.
2. Aba **Sign-in method** → clique em **E-mail/senha** → **Ativar** → **Salvar**.

## Passo 3 — Criar o banco (Firestore)
1. Menu esquerdo: **Criação → Firestore Database → Criar banco de dados**.
2. Escolha o modo **produção** e a região (ex.: `southamerica-east1`). **Criar**.

## Passo 4 — Colar as regras de segurança
1. Ainda no Firestore, aba **Regras (Rules)**.
2. Apague o que estiver lá e cole **exatamente** isto, depois **Publicar**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isAdmin() {
      return request.auth != null
        && exists(/databases/$(database)/documents/users/$(request.auth.uid))
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    match /users/{uid} {
      allow read:  if request.auth != null && (request.auth.uid == uid || isAdmin());
      allow write: if isAdmin();
    }
  }
}
```

Isso garante: cada pessoa só lê o próprio perfil; **só o admin** cria/edita/exclui acessos.

## Passo 5 — Pegar a configuração do app (config web)
1. Menu: **Visão geral do projeto** (engrenagem) → **Configurações do projeto**.
2. Role até **Seus apps** → clique no ícone **`</>` (Web)**.
3. Apelido: `dashboard` → **Registrar app**.
4. Vai aparecer um bloco `const firebaseConfig = { ... }`. **Copie só o objeto** `{ ... }`
   (apiKey, authDomain, projectId, appId, etc.).

> A `apiKey` do app web **não é secreta** — pode ficar no arquivo sem problema. A segurança
> vem das **Regras** (Passo 4) e do login, não de esconder a config.

## Passo 6 — Ligar a config no dashboard
Escolha **uma** das opções:

**Opção A (recomendada — vale para todos):** abra o `index.html` num editor de texto,
ache o trecho `let FIREBASE_CONFIG = {` (perto do fim) e substitua pelos seus valores:

```js
let FIREBASE_CONFIG = {
  apiKey: "AIza…",
  authDomain: "arca-dashboard.firebaseapp.com",
  projectId: "arca-dashboard",
  appId: "1:…:web:…"
};
```

Salve o arquivo. Pronto — agora ele pede login.

**Opção B (rápida, por dispositivo):** abra o dashboard, clique em **Configurar servidor
(Firebase)** na tela de login, cole o objeto de config e clique em **Salvar e recarregar**.

> Se preferir, me mande esse objeto de config que eu já deixo colado no arquivo pra você.

## Passo 7 — Criar o PRIMEIRO admin (manual, só uma vez)
Como só um admin pode criar outros acessos, o primeiro precisa ser criado na mão:

1. **Authentication → Users → Adicionar usuário**: coloque o **e-mail** e uma **senha**. Criar.
2. Copie o **UID** desse usuário (aparece na lista de Users).
3. **Firestore Database → Iniciar coleção** → ID da coleção: **`users`**.
4. **ID do documento**: cole o **UID** copiado. Adicione os campos:
   - `name` (string): seu nome — ex.: `Administrador`
   - `email` (string): o mesmo e-mail
   - `role` (string): **`admin`**
   - `expert` (string): deixe vazio
   - `active` (boolean): **true**
5. Salvar.

## Passo 8 — Entrar e criar os demais acessos
1. Abra o `index.html`, entre com o e-mail/senha do admin.
2. Vá na aba **Admin** → **Criar novo acesso**. Para cada pessoa escolha o **Perfil**:
   - **Administrador** — vê tudo e gerencia acessos.
   - **Expert** — vê **só a própria performance** (informe o *Expert vinculado* com o nome
     **exatamente igual** ao que está na coluna `Expert` das planilhas, ex.: `Expert 1`).
   - **Comercial** — vê **só a aba Time Comercial**.
3. Na lista você pode **Desativar/Ativar**, **enviar reset de senha** (vai um e-mail para a
   pessoa) ou **Excluir** o acesso.

---

## (Opcional) Publicar como site com link
Para a equipe acessar por um link (em vez de um arquivo), dá pra hospedar de graça:
Firebase Hosting, Netlify ou GitHub Pages. Se quiser, eu te ajudo a publicar — aí o
endereço já fica autorizado no Firebase automaticamente (Firebase Hosting).

## ⚠️ Importante sobre a segurança dos dados
O login protege o **acesso ao painel** e cada perfil só vê o que deve. Mas hoje o dashboard
lê as **Planilhas do Google direto no navegador** — então, para ele funcionar, os **links das
planilhas precisam continuar acessíveis**. Ou seja: o login protege o *painel*, e o filtro por
perfil acontece no navegador. Para proteção **total** dos dados brutos (esconder as linhas de
outros experts mesmo de quem abrir o "inspecionar"), o próximo passo seria eu montar um
pequeno servidor (Cloud Function) que lê as planilhas e entrega só o permitido. Me avise se
quiser evoluir para esse nível — por enquanto, para uso interno, o login + perfis já organiza
e restringe bem o acesso.
