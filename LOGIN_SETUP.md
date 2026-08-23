# 🔒 Configurar login no Firebase (passo a passo)

O código do app já está pronto e publicado (assim que o GitHub Pages atualizar). Faltam só os passos abaixo, direto no [console do Firebase](https://console.firebase.google.com) — são coisas que só o dono do projeto consegue fazer, eu não tenho acesso a esse painel.

## Passo 1 — Habilitar o login por Email/Senha

1. Abra o [console do Firebase](https://console.firebase.google.com) → projeto **orcamento-familia-7dccf**
2. Menu lateral → **Authentication** → aba **Sign-in method**
3. Clique em **Email/Password** → ative o primeiro toggle (**Email/Password**) → **Salvar**

## Passo 2 — Criar uma conta pra cada pessoa

O app pede só **Nome** e **Senha** na tela de login, mas por trás ele monta um email fictício automaticamente: o nome, sem espaços/acentos/maiúsculas, seguido de `@familia-delira.app`. Exemplos:
- Nome digitado `Diego` → email real criado: `diego@familia-delira.app`
- Nome digitado `Ana Paula` → email real criado: `anapaula@familia-delira.app`

Pra cada pessoa que vai usar o app:

1. Ainda em **Authentication**, aba **Users** → **Add user**
2. **Email**: o email fictício correspondente ao nome que essa pessoa vai digitar (veja acima)
3. **Password**: qualquer senha forte — é o que a pessoa vai digitar no app
4. **Add user**

Repita para quantas pessoas precisar (você, sua esposa, etc.).

## Passo 3 — Copiar o UID de cada conta

Na mesma lista de **Users**, cada linha mostra uma coluna **User UID** — uma string tipo `a1B2c3D4e5F6...`. Copie o UID de cada conta que você criou no Passo 2. Vai precisar deles no próximo passo.

## Passo 4 — Travar as Regras do banco

1. Menu lateral → **Realtime Database** → aba **Regras** (Rules)
2. **Substitua tudo** que estiver lá por isto, trocando `UID_DIEGO` e `UID_ESPOSA` pelos UIDs reais que você copiou (adicione mais linhas com `||` se tiver mais gente):

```json
{
  "rules": {
    "inbox": {
      ".write": "auth != null"
    },
    ".read": "auth.uid === 'UID_DIEGO' || auth.uid === 'UID_ESPOSA'",
    ".write": "auth.uid === 'UID_DIEGO' || auth.uid === 'UID_ESPOSA'"
  }
}
```

3. **Publicar** (Publish)

**Por que o `inbox` é diferente:** é a caixa de entrada que a automação do Atalho do iPhone usa pra lançar gastos do Apple Pay sozinha — ela usa login anônimo, não o login de vocês. Deixei só a *escrita* ali aberta pra qualquer autenticado (a leitura continua travada só pra vocês dois) — assim a automação continua funcionando, e ninguém de fora consegue ler nada, só empurrar um lançamento pra fila que só vocês processam.

## Passo 5 — Testar

1. Abra o app publicado (força um recarregamento — Ctrl+F5 ou feche e abra de novo no iPhone)
2. Deve aparecer a tela de login. Digite o nome e a senha que você cadastrou no Passo 2
3. Se funcionar, o app abre normal — e a partir daí fica logado nesse aparelho até você tocar no 🚪 (sair) na barra do topo

## Se der errado

- **"Nome ou senha incorretos" mesmo com os dados certos** → confira se o Passo 1 (habilitar Email/Password) foi salvo, e se o email fictício bate exatamente com o que você cadastrou no Passo 2 (sem espaço, sem acento, tudo minúsculo, antes do `@familia-delira.app`).
- **App trava em "Conectando..." depois do login** → provavelmente as Regras do Passo 4 não foram publicadas, ou o UID foi copiado errado. Confira se o UID no console bate exatamente com o que está nas Regras.
- **A automação do Atalho parou de lançar gastos** → confira se a regra do `inbox` ficou exatamente como no Passo 4 (a escrita ali precisa continuar em `"auth != null"`, separada da regra geral).
