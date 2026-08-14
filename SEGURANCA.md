# Trancar o banco de dados do dashboard

Hoje qualquer pessoa com o endereço do banco lê **tudo**: salário de todo mundo,
quanto cada cliente paga, resultado de cada mês. O endereço está dentro do código
da página, que é pública. E a regra de escrita aceita login anônimo, então em tese
dá pra apagar os 26 meses de histórico.

O código do painel já está pronto pro mundo trancado. Falta só apertar 3 botões no
console do Firebase. São eles que só você pode apertar, porque mexem na conta.

Link do console: https://console.firebase.google.com/project/abrii-dashboard

---

## Passo 1 — ligar o login por e-mail e senha

1. No menu da esquerda: **Build → Authentication**
2. Aba **Sign-in method**
3. Clicar em **Email/Password** → ligar a primeira chavinha (**Enable**) → **Save**

## Passo 2 — criar os usuários

Ainda em **Authentication**, aba **Users** → botão **Add user**:

| E-mail | Pra que serve | Senha |
|---|---|---|
| `abriiiacademy@gmail.com` | você, com permissão de editar | escolha uma forte |
| `socios@abriiaceleradora.com.br` | sócios, só olham | outra senha |

> O e-mail do dono precisa ser **exatamente** `abriiiacademy@gmail.com`, porque é
> ele que está na lista `ADMIN_EMAILS` do painel e nas regras abaixo. Se quiser usar
> outro, troque nos dois lugares.

Qualquer e-mail que não seja o seu entra em modo somente-leitura automaticamente.

## Passo 3 — publicar as regras

Menu da esquerda: **Build → Realtime Database** → aba **Rules**.
Apagar o que está lá e colar isto:

```json
{
  "rules": {
    "abrii": {
      ".read": "auth != null && auth.token.firebase.sign_in_provider === 'password'",
      ".write": "auth != null && auth.token.email === 'abriiiacademy@gmail.com'"
    },
    "backups": {
      ".read": "auth != null && auth.token.email === 'abriiiacademy@gmail.com'",
      ".write": "auth != null && auth.token.email === 'abriiiacademy@gmail.com'"
    },
    "backupsIdx": {
      ".read": "auth != null && auth.token.email === 'abriiiacademy@gmail.com'",
      ".write": "auth != null && auth.token.email === 'abriiiacademy@gmail.com'"
    },
    "publico": {
      ".read": true,
      ".write": "auth != null && auth.token.email === 'abriiiacademy@gmail.com'"
    }
  }
}
```

Clicar em **Publish**.

O que cada bloco faz:

- **abrii**: o financeiro inteiro. Só lê quem entrou com e-mail e senha. Login anônimo
  não passa mais, e é justamente por ele que hoje o banco está aberto. Só você escreve.
- **backups**: os snapshots diários. Só você.
- **publico**: só o total de cada mês, sem descrição, sem salário, sem nome de cliente.
  Fica aberto de propósito, porque é dele que o planejamento 2026 se alimenta.
- Tudo que não está na lista fica bloqueado por padrão.

## Passo 4 — entrar no painel

Abrir https://railanjunior.github.io/abrii-dashboard/ e entrar com o e-mail e a senha
do passo 2. Se aparecer "modo legado" na barra lateral, clicar em **Sair** e entrar de novo.

## Passo 5 — as notificações do Mac

O script que avisa às 8h e às 13h também precisa da senha agora. Criar o arquivo:

```bash
cp ~/Library/Application\ Support/abrii-notify/credenciais.exemplo.json ~/Library/Application\ Support/abrii-notify/credenciais.json
```

Abrir o `credenciais.json`, trocar a senha pela que você criou, salvar, e travar o arquivo:

```bash
chmod 600 ~/Library/Application\ Support/abrii-notify/credenciais.json
```

Testar:

```bash
python3 ~/Library/Application\ Support/abrii-notify/abrii_notify.py
```

Tem que aparecer `login no Firebase ok` no log (`/tmp/abrii-notify.log`).

---

## Como conferir que ficou trancado

Depois de publicar as regras, rodar no terminal:

```bash
curl -s -o /dev/null -w "%{http_code}\n" "https://abrii-dashboard-default-rtdb.firebaseio.com/abrii/raw.json"
```

- **401** = trancado, era isso que a gente queria.
- **200** = ainda aberto, alguma coisa não foi publicada.

E o público (esse tem que continuar 200, é o que o planejamento consome):

```bash
curl -s -o /dev/null -w "%{http_code}\n" "https://abrii-dashboard-default-rtdb.firebaseio.com/publico/meses.json"
```

---

## Se der problema

O painel tem uma saída de emergência: na tela de login existe o link cinza
**"Ainda nao configurei o login no Firebase"**, que volta pro modo antigo. Ele só
funciona enquanto as regras estiverem abertas, então serve pra transição, não pra
ficar. Depois de trancar, esse caminho mostra o painel vazio, e aí é sinal de que
você precisa entrar com e-mail e senha mesmo.

Os dados nunca somem: ficam no localStorage do navegador, no backup diário do
Firebase e no botão **Exportar HTML**.
