# Funil de Experts — como ligar e como funciona

Board de 6 etapas dentro do dashboard (aba **Funil de Experts**). Cada expert é um card que
percorre as etapas, sempre com **um dono único**, e fica marcado em vermelho quando passa do
prazo da etapa.

Os dados do funil **não vêm das planilhas** — moram no Firestore, no mesmo projeto Firebase
que já é usado no login.

---

## 1. Ligar (uma vez só)

### Passo 1 — Publicar as regras de segurança

Sem isto o funil não abre. **Firebase → Firestore Database → Regras**, apague tudo e cole:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function perfil()  { return get(/databases/$(database)/documents/users/$(request.auth.uid)).data; }
    function logado()  { return request.auth != null
                            && exists(/databases/$(database)/documents/users/$(request.auth.uid))
                            && perfil().active != false; }
    function isAdmin() { return logado() && perfil().role == 'admin'; }
    function isEquipe(){ return logado() && (perfil().role == 'admin' || perfil().role == 'equipe'); }

    match /users/{uid} {
      allow read:  if request.auth != null && (request.auth.uid == uid || isAdmin());
      allow write: if isAdmin();
    }

    // prazos por etapa + lista de nichos
    match /config/{doc} {
      allow read:  if isEquipe();
      allow write: if isAdmin();
    }

    // quem revelou credencial de qual expert e quando — imutável
    match /credLog/{id} {
      allow read, create:   if isAdmin();
      allow update, delete: if false;
    }

    match /experts_funil/{cardId} {
      allow read, create, update: if isEquipe();
      allow delete:               if isAdmin();

      // LOGIN/SENHA DA ZONA DE JOGO — só admin, nem leitura para os demais
      match /secure/{doc} {
        allow read, write: if isAdmin();
      }

      match /historico/{id} {                 // trilha de auditoria — não se apaga
        allow read, create:   if isEquipe();
        allow update, delete: if false;
      }
      match /relatorios/{id} {
        allow read, create: if isEquipe();
        allow update, delete: if isAdmin();
      }
      match /anexos/{id} {
        allow read, write: if isEquipe();
      }
    }
  }
}
```

Clique em **Publicar**.

> Estas regras **substituem** as do `COMO-ATIVAR-LOGIN.md` (elas já incluem o bloco de
> `users` que estava lá). Cole este conjunto inteiro, não os dois.

### Passo 2 — Dar acesso à equipe

Na aba **Admin** do dashboard, para cada pessoa:

| Pessoa | Perfil | Pessoa no funil |
|---|---|---|
| Fábio | Administrador | Fábio |
| Mizag | Administrador | Mizag |
| Márcio | Administrador | Márcio |
| Camily | Equipe interna | Camily |
| William | Equipe interna | William |

- **Administrador** — vê tudo, gerencia acessos e é o único que revela credenciais.
- **Equipe interna** — vê **só** a aba Funil de Experts. Não vê financeiro nem comercial.
- O campo **Pessoa no funil** é o que faz os cards caírem no nome da pessoa. Sem ele a pessoa
  vê o board, mas nenhum card aparece como dela. Dá para ajustar depois pela coluna
  "Pessoa no funil" na lista de acessos.

Experts e o time comercial continuam sem acesso nenhum ao funil.

---

## 2. As 6 etapas

| # | Etapa | Dono | Sai daqui quando | Prazo padrão |
|---|---|---|---|---|
| 1 | Entrada / Boas-vindas | Camily | Dados do expert captados | mesmo dia |
| 2 | Organização | Camily | Card, grupo, contrato e credenciais registrados | 1 dia |
| 3 | Alinhamento da operação | William | Descrição da operação documentada | 1 dia |
| 4 | Criação da LP | Márcio | LP pronta | 2 dias |
| 5 | Publicação | William | Tráfego ativo | 3 dias |
| 6 | Rodando | William (apoio Fábio/Mizag) | — (etapa final) | relatório a cada 7 dias |

A movimentação é **manual**: a pessoa abre o card e clica em **Concluir etapa**. Ao mudar de
etapa o dono troca sozinho — nenhum card fica órfão.

**Os prazos são editáveis no próprio painel** (botão *Prazos e nichos*, só admin). Os valores
acima são só o ponto de partida; ajuste com a equipe sem precisar mexer no código.

### O que o sistema cobra antes de deixar concluir

| Etapa | Exige |
|---|---|
| 1 | nome, nicho e data de entrada |
| 2 | link do card/grupo, contrato (link ou observação) e credenciais cadastradas |
| 3 | a descrição de como o expert opera hoje |
| 4 | link de destino do botão da LP e grupo marcado como "pronto" |

Se faltar algo, o painel diz exatamente o que é. Administrador pode forçar a passagem, mas
recebe o aviso antes e precisa confirmar.

---

## 3. Alerta de atraso

- **Borda e selo vermelhos** — passou do prazo da etapa.
- **Borda âmbar, "vence hoje"** — está no último dia do prazo.
- Na etapa 6 o alerta é outro: dispara quando passa do intervalo sem relatório semanal.
- O KPI **Atrasados** no topo mostra o total, e o filtro **Só atrasados** isola esses cards.

O prazo conta **dias corridos parados na etapa atual** (não desde a entrada do expert).
Prazo `0` significa "tem que sair no mesmo dia".

---

## 4. Credenciais da zona de jogo

Este é o campo sensível e ele é tratado à parte:

- Fica numa **subcoleção separada** (`experts_funil/{card}/secure/credenciais`), **não** no
  documento do card que a equipe toda lê.
- As regras acima liberam essa subcoleção **só para perfil admin**. Não é só a tela que
  esconde — quem não é admin não consegue ler o dado nem pelo console do navegador.
- Na tela aparece mascarado (`••••`). Revelar exige um clique.
- **Toda revelação vira registro** em `credLog` (quem, e-mail, qual expert, data e hora).

**O que isto não é:** não é criptografia ponta a ponta. Quem for administrador — ou tiver
acesso ao console do Firebase — lê os valores. Para esconder até do administrador do banco
seria preciso uma senha-mestra digitada pela equipe, com o risco de perder as credenciais se
a senha se perder. Se quiserem esse nível, é uma mudança pequena — é só decidir.

---

## 5. Notificações

Hoje o aviso é **dentro do painel**:

- **Sino no topo** com a contagem dos seus cards atrasados, vencendo hoje ou que acabaram de
  cair no seu nome. Clicar em um item abre o card direto.
- **Aviso flutuante em tempo real**: com o painel aberto, quando um card passa a ser seu, a
  notificação aparece na hora (e vira notificação do sistema, se você autorizar o navegador).
- KPI **Meus cards**, e filtro **Só os meus**.

**WhatsApp e e-mail ainda não saem** — isso precisa de um servidor que guarde as chaves da
API (o site é estático, uma chave no `index.html` ficaria pública). O caminho natural é uma
Cloudflare Pages Function com as chaves nas variáveis de ambiente, ou um webhook para
Zapier/Make. É um passo separado; me avise quando quiser ligar.

---

## 6. Anexos e fotos

Contrato e fotos do expert podem ir por **link** (Drive, Dropbox) ou **upload direto**.

No upload, as imagens são reduzidas no navegador antes de salvar e ficam guardadas no próprio
Firestore — assim não é preciso ativar o Firebase Storage (que exige plano pago). O limite é
~700 KB por arquivo; acima disso, use o campo de link.

---

## 7. Onde os dados ficam

```
experts_funil/{cardId}                  dados do card, etapa, responsável, datas
experts_funil/{cardId}/secure/credenciais   login e senha da zona de jogo (só admin)
experts_funil/{cardId}/historico/{id}   toda mudança de etapa e troca de dono (imutável)
experts_funil/{cardId}/relatorios/{id}  relatórios semanais da etapa 6
experts_funil/{cardId}/anexos/{id}      contrato e fotos enviadas
config/funil                            prazos por etapa e lista de nichos
credLog/{id}                            quem revelou credencial de quem, e quando
users/{uid}                             acessos (já existia) + campo "pessoa" do funil
```

Arquivar um card só o tira do board — histórico, relatórios e anexos continuam guardados, e
o botão **Ver arquivados** traz de volta.

---

## 8. Ainda em aberto

Os dois pontos que você marcou como pendentes ficaram **configuráveis no painel**, então não
travam a entrega — mas vale fechar com a equipe:

1. **Lista de nichos** — começa com: apostas esportivas, cassino ao vivo, roleta,
   aviator/crash, slots, poker, bingo, e-sports, outro. Edite em *Prazos e nichos*.
2. **Prazo de cada etapa** — os padrões da tabela acima vieram do que você definiu (captar no
   mesmo dia, LP em 2 dias, tráfego em 3). As etapas 2 e 3 ficaram com 1 dia como chute
   inicial e precisam do aval do time.
