# 🤖 Lançamento automático de pagamentos Apple Pay (iPhone)

Toda vez que você **pagar por aproximação com o iPhone ou Apple Watch**, o gasto é lançado sozinho no app de orçamento em segundos — com categoria automática e o selo 🤖 para você identificar (e ajustar a categoria se quiser).

**O que é coberto:** pagamentos por aproximação com cartões cadastrados na Carteira (Apple Pay).
**O que NÃO é coberto (limitação da Apple):** PIX, compras dentro de apps de banco e cartão físico. Para esses, continue usando o lançamento manual ou a importação de extrato/fatura — a detecção de duplicatas evita contar duas vezes o que já entrou automaticamente.

## Requisitos

- iOS 17 ou mais novo (no iOS 26 o gatilho mudou de nome: **"Transação" → "Carteira"**)
- Cartão(ões) cadastrado(s) na Carteira / Apple Pay

## Passo 1 — Criar a automação

1. Abra o app **Atalhos** → aba **Automação** → **Nova Automação** (+)
2. Escolha o gatilho **Transação** (ou **Carteira**, no iOS 26)
3. Em **Cartão ou Passe**, selecione os cartões que você usa no dia a dia (pode marcar todos)
4. Deixe Categoria e Comerciante como "Qualquer"
5. Marque **Executar Imediatamente** (para não pedir confirmação a cada compra)
6. Toque em **Avançar** → **Nova Automação em Branco**

## Passo 2 — Adicionar as ações (nesta ordem)

**Ação 1 · Formatar Data**
- Busque a ação **"Formatar Data"**
- Data: **Data Atual** · Formato: **Personalizado** · Texto do formato: `yyyy-MM-dd`

**Ação 2 · Obter Tempo entre Datas**
- Busque **"Obter tempo entre datas"**
- De: digite `01/01/1970` · Até: **Data Atual** · Em: **Segundos**

**Ação 3 · Obter Conteúdo do URL** (autenticação)
- URL: `https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=AIzaSyBYq3CiQmvD0YXbJNGL-ac4nPc8NoSkh-w`
- Toque na seta para expandir → Método: **POST** · Corpo do Pedido: **JSON**
- Adicione 1 campo: chave `returnSecureToken` · tipo **Booleano** · valor **verdadeiro**

**Ação 4 · Obter Valor do Dicionário**
- Busque **"Obter Valor do Dicionário"**
- Obter **Valor** de **`idToken`** em **Conteúdo do URL** (resultado da Ação 3)

**Ação 5 · Obter Conteúdo do URL** (envia o lançamento)
- URL: `https://orcamento-familia-7dccf-default-rtdb.firebaseio.com/inbox.json?auth=` e logo após o `=` insira a variável **Valor do Dicionário** (da Ação 4)
- Método: **POST** · Corpo do Pedido: **JSON**, com estes campos (todos tipo **Texto**):

| Chave | Valor |
|-------|-------|
| `valor` | variável **Quantia** (toque em "Entrada do Atalho" → propriedade **Quantia/Amount**) |
| `desc` | variável **Comerciante** (Entrada do Atalho → **Comerciante/Merchant**) |
| `data` | variável **Data Formatada** (resultado da Ação 1) |
| `ts` | variável **Tempo Entre Datas** (resultado da Ação 2) |
| `pgto` | `Apple Pay` (texto fixo) |

7. Toque em **OK/Concluído** para salvar a automação.

## Passo 3 — Testar

Faça qualquer compra por aproximação (vale recarga de bilhete, café etc.). Em segundos, abra o app de orçamento: o gasto aparece em **Lançamentos** com o selo 🤖. Se o app estiver aberto, aparece o aviso "🤖 Lançado: …".

Alternativa sem gastar: crie um **atalho normal** (aba Atalhos) só com as Ações 3, 4 e 5, preenchendo `valor` = `1,00` e `desc` = `TESTE`, e execute. Depois apague o lançamento de teste no app (🗑).

## Solução de problemas

- **Erro 401 / Permission denied na Ação 5** → as regras do Realtime Database estão bloqueando. No [console do Firebase](https://console.firebase.google.com) → Realtime Database → Regras, garanta que usuários autenticados podem escrever em `inbox`, por exemplo: `"inbox": { ".write": "auth != null", ".read": "auth != null" }`
- **Automação não dispara** → confira iOS 17+, se o cartão usado está selecionado no gatilho e se "Executar Imediatamente" está ativo.
- **Valor errado (ex.: 4590 em vez de 45,90)** → confira se o campo `valor` está recebendo a variável Quantia, sem formatação extra.
- **Lançou em categoria errada** → normal; toque em ✏️ no lançamento e corrija. A categorização automática usa palavras-chave do nome do estabelecimento.

## Como funciona por dentro

O Atalho faz um login anônimo no Firebase do app (Ação 3–4) e grava o pagamento numa "caixa de entrada" (`inbox`). Qualquer aparelho com o app aberto processa essa caixa: categoriza pelo nome do estabelecimento, cria o lançamento (idempotente — dois aparelhos processando ao mesmo tempo não duplicam) e limpa a caixa. Nada sai do seu próprio Firebase.
