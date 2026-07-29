# Hub de Catálogos — com preços

Portal em `catalogosdeprecos.chumbada.com.br`. Reúne os 5 catálogos da Chumbada Oficial
na versão **com preços** (consulta para lojistas e equipe comercial) e oferece uma busca
global de produtos que leva direto ao item dentro do catálogo certo.

O hub equivalente **sem preços** é [`hub-catalogos-sem-preco`](https://github.com/mlxjack/hub-catalogos-sem-preco)
(`catalogos.chumbada.com.br`).

## Catálogos ligados a este hub

| Catálogo | Domínio | Repositório |
| --- | --- | --- |
| Anzóis & Jig Heads | `precosdosanzois.chumbada.com.br` | `catalogo-de-anzois-e-jigs-com-preco-v2` |
| Iscas | `precodasiscas.chumbada.com.br` | `catalogo-iscas-v2` |
| Acessórios | `precodosacessorios.chumbada.com.br` | `catalogo-acessorios-precos-chumbada` |
| Chumbadas & Jig Heads | `precodaschumbadas.chumbada.com.br` | `catalogo-de-chumbadas-e-jig-heads-com-preco` |
| Óculos | `precodosoculos.chumbada.com.br` | `oculos-chumbada-oficial` |

## Busca global

`index.html` carrega `search-index.json` e faz a busca no navegador — sem
servidor e sem depender dos catálogos em tempo de execução. A busca ignora
acentos (`oculos` acha `Óculos`), exige que todos os termos apareçam e aceita
nome, categoria, cor, peso ou SKU.

Cada resultado aponta para o produto específico:

| Catálogo | Formato do link |
| --- | --- |
| Anzóis, Iscas | `<domínio>/#/product/<handle>` |
| Acessórios, Chumbadas | `<domínio>/#/produto/<slug>` |
| Óculos | `<domínio>/#/produto/<sku>` |

Se `search-index.json` não carregar, a seção de busca é removida da página e os
cards de catálogo continuam funcionando normalmente.

## Regerar o índice

**Toda vez que um catálogo mudar de produtos, o índice precisa ser regerado** —
ele é um retrato estático, não uma consulta ao vivo.

O script espera os repositórios dos catálogos clonados **como pastas irmãs** desta:

```bash
node tools/build-search-index.mjs
```

Saída esperada:

```
Variante: com-preco (preços visíveis)

  ✓ anzois        44 produtos
  ✓ iscas         85 produtos
  ✓ acessorios    79 produtos
  ✓ chumbadas    126 produtos
  ✓ oculos        38 produtos

372 produtos → search-index.json
```

Se algum catálogo falhar, o script **aborta sem escrever** o índice — assim uma
pasta ausente nunca gera um índice pela metade. Depois é só commitar o
`search-index.json` atualizado.

### Como os produtos são lidos

Cada catálogo guarda seus dados de um jeito diferente, e o script respeita isso:

- **Anzóis e Iscas** — carrega o `src/utils/csvParser.js` do próprio catálogo e
  apenas troca a leitura via HTTP por leitura local do `products.csv`. Isso é
  proposital: o parser de Anzóis agrupa jig heads por modelo e deriva o slug com
  `slugify(modelTitle)`, então usar o `Handle` cru do CSV geraria links quebrados.
  Reaproveitar o parser garante que os slugs do índice sejam sempre os mesmos que
  o catálogo usa.
- **Acessórios** — avalia `assets/js/data.js` e lê `window.PRODUCTS`.
- **Chumbadas e Óculos** — extrai o array `PRODUCTS` / `produtos` embutido no `index.html`.

Se um desses parsers mudar de forma incompatível, o script falha com mensagem
explícita em vez de gerar um índice silenciosamente errado.
