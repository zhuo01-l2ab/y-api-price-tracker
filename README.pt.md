# Tabela de preços da Y-API — recalculada, não copiada

[English](README.md) · [简体中文](README.zh.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Deutsch](README.de.md) · **Português**

> **Divulgação:** eu trabalho na Y-API, então esta é uma ferramenta de primeira parte lendo um arquivo de primeira parte. Tudo o que ela imprime vem de <https://y-api.bestvirtualgoods.com/pricing.json>, que é público e não exige chave de API — rode o script e confira a saída contra a fonte você mesmo.

Um script, sem dependências, sem chave de API. Ele lê o arquivo de preços publicado pela Y-API e gera [table.md](table.md) com preços em **dinheiro** — o valor que realmente sai do seu cartão, e o único comparável ao preço de tabela de outro fornecedor.

## Por que existe

A Y-API cobra em **crédito**, e hoje $1 pago vira $20 em crédito. Crédito é o que é descontado do saldo; não é o que é cobrado do seu cartão. Uma tabela que para no crédito faz cada modelo parecer 20× mais caro, e uma tabela com "dividir por 20" hardcoded fica silenciosamente errada no dia em que a taxa promocional acabar. Este script lê a taxa do **mesmo arquivo** de onde lê os preços.

## Como rodar

```bash
node price-table.mjs > table.md                            # busca o arquivo publicado
PRICE_JSON=./pricing.json node price-table.mjs             # renderiza uma cópia salva, offline
```

Requer Node 18+ (usa o `fetch` global). O `table.md` deste repositório é regerado toda segunda-feira por uma GitHub Action e **só é commitado quando muda** — ou seja, o histórico de commits *é* o registro de mudanças de preço.

## Duas regras que o script mantém

Quebrar qualquer uma delas produz uma tabela que contradiz silenciosamente o site de onde ela veio.

1. **O preço em dinheiro é recalculado, não copiado.** O arquivo publica `cash_price`, mas o script recalcula `credit_price / top_up.quota_rate` e **falha alto** se os dois divergirem. Nem uma taxa vencida nem um número editado à mão conseguem entrar na tabela.
2. **`vendor_cheaper_on_cached_input` nunca é impresso sozinho.** Essa flag diz que a tarifa de entrada em cache do fornecedor está abaixo do nosso preço em dinheiro — somos a opção mais cara para uma carga pesada em cache. Porém algumas linhas a carregam junto com `historical-price` (uma tabela antiga do fornecedor), e o site da Y-API as exclui deliberadamente da sua contagem de "perdemos no cache". **Imprimir a flag sem a ressalva faz o leitor sair com um número errado.** Por isso a tabela traz as duas colunas.

## Como ler a saída

- **O catálogo** — todos os modelos, crédito e dinheiro, por 1M de tokens, ordenados por preço.
- **Contra o preço de tabela do fornecedor** — os 11 modelos com preço de fornecedor que verificamos e citamos, com o múltiplo, a flag de cache, as ressalvas, a data de verificação e o link para a página do fornecedor. Os 4 restantes aparecem como sem preço de fornecedor verificável; o arquivo diz isso explicitamente em vez de estimar.

`× ours` se lê como "o fornecedor cobra este múltiplo do nosso preço em dinheiro" — um múltiplo grande significa que **a Y-API é mais barata**, não mais cara.

## Escopo, com honestidade

- A tabela é tão atual quanto o arquivo fonte. Ela traz o `synced_at` da fonte e a data de renderização no cabeçalho; se ambos estiverem velhos, não há novidade aqui.
- Ela renderiza os preços de **um** gateway. Não compara entre fornecedores nem acompanha o histórico de preços de terceiros.
- Os preços mudam, e a taxa de recarga é promocional, sem data de término anunciada. Quem decidir algo a partir desta tabela deve recalcular a partir da fonte, não de uma cópia do `table.md`.

---

*Parte do [perfil da Y-API](https://github.com/zhuo01-l2ab).*
