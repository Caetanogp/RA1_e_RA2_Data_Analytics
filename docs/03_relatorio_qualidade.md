# Relatório de Avaliação da Qualidade dos Dados

Investigação completa em `notebooks/02_avaliacao_qualidade.ipynb`. Este documento resume os achados e as decisões de tratamento aplicadas em `notebooks/03_limpeza_preparacao.ipynb`.

## Tipos de dados incorretos

Todas as colunas de data/hora (`orders`, `items`, `reviews`) chegam da ingestão como texto. **Tratamento:** convertidas para `datetime`.

Os prefixos de CEP (`customer_zip_code_prefix`, `seller_zip_code_prefix`, `geolocation_zip_code_prefix`) chegam como número inteiro, mas são identificadores categóricos - vários prefixos reais começam com `0` (região de São Paulo). **Tratamento:** convertidos para texto com 5 dígitos (zero-padded).

## Valores ausentes

| Tabela | Coluna(s) | Qtde. | Natureza | Decisão |
|---|---|---|---|---|
| orders | `order_approved_at`, `order_delivered_carrier_date`, `order_delivered_customer_date` | 160 / 1.783 / 2.965 | Legítima - pedido não chegou a essa etapa (cancelado/indisponível/em processamento) | Mantido como `NaT` |
| reviews | `review_comment_title`, `review_comment_message` | 87.656 / 58.247 | Legítima - comentário é opcional | Mantido como `NaN` |
| products | `product_category_name` + metadados associados | 610 | Legítima - cadastro incompleto do vendedor | Mantido como `NaN` |
| products | dimensões físicas | 2 | Legítima - cadastro incompleto | Mantido como `NaN` |

Em nenhum caso os ausentes foram imputados: todos correspondem a uma ausência real de informação, e preencher o valor fabricaria dado que não existe.

## Registros duplicados

`olist_geolocation_dataset.csv` tinha 261.831 linhas 100% duplicadas (~26% do arquivo) - mesmo CEP, mesma lat/lng, mesma cidade/estado, sem nenhuma informação adicional. **Tratamento:** removidas (`drop_duplicates`).

As demais 8 tabelas não tinham nenhuma duplicata de linha inteira.

`review_id` duplicado (814 ocorrências) **não é tratado como duplicata**: o par `(review_id, order_id)` é único - cada linha é a avaliação real de um pedido específico; é o identificador de review que é reaproveitado entre pedidos diferentes (particularidade conhecida do dataset da Olist). Documentado para não ser usado como chave primária.

## Valores inválidos ou impossíveis

| Tabela | Problema | Qtde. | Decisão |
|---|---|---|---|
| payments | `payment_type == "not_defined"` com `payment_value == 0` | 3 | Removidas (não representam pagamento real, sem forma de inferir o tipo correto) |
| payments | `payment_installments == 0` com valor > 0 | 2 | Corrigido para `1` (zero parcelas é impossível havendo pagamento) |
| products | `product_weight_g == 0` | 4 | Convertido para `NaN` (peso zero é fisicamente impossível) |

`payment_value == 0` em linhas `voucher` (6 casos) **foi mantido**: são parcelas de um pagamento fracionado em múltiplos vouchers, não um erro.

## Categorias inconsistentes

Estados (`customer_state`, `seller_state`, `geolocation_state`) estão 100% dentro das 27 UFs válidas - nenhuma correção necessária.

`geolocation_city` tinha qualidade bem inferior a `customer_city`/`seller_city`: ~20 valores claramente inválidos - 4 em que o "nome da cidade" era na verdade uma sigla de estado (`sp`, `rj`), e ~16 com texto corrompido (entidades HTML, trechos de URL-encoding, pontuação solta). **Tratamento:** após a remoção das duplicatas, 16 desses valores restaram e foram convertidos para `NaN`.

**Limitação documentada e aceita:** além desses casos claramente inválidos, `geolocation_city` tem milhares de variações de grafia de cidade (com/sem acento, espaçamento) que não foram canonicalizadas - isso exigiria uma tabela de referência externa de municípios brasileiros, fora do escopo do RA1. Quem precisar de granularidade de cidade de forma confiável deve preferir `customer_city`/`seller_city` (já validados) ou os prefixos de CEP.

`product_category_name`: 2 categorias (`pc_gamer`, `portateis_cozinha_e_preparadores_de_alimentos`) não têm entrada correspondente em `product_category_name_translation.csv` (13 produtos afetados). **Tratamento:** mantidas com `product_category_name_english` nulo após a junção - lacuna da tabela de tradução original, documentada e não inventada.

## Outliers

31 das ~1 milhão de linhas de `olist_geolocation_dataset.csv` tinham latitude/longitude fora da caixa delimitadora aproximada do território brasileiro (lat entre -34 e 6, lng entre -74 e -32) - erros de geocodificação incompatíveis com um dataset 100% de pedidos no Brasil. **Tratamento:** removidas - 27 deles restaram após a remoção das duplicatas, e esses foram os removidos na limpeza.

Nenhum outlier foi identificado em `price`, `freight_value` ou `payment_value` que justificasse remoção: os valores máximos observados (ex. pedido de R$ 6.735, frete de R$ 409) são compatíveis com produtos de alto valor/grande porte e não foram tratados como erro.

## O que não foi tratado (e por quê)

- Variações de grafia de nomes de cidade em `geolocation_city` (ver seção de categorias).
- 2 categorias de produto sem tradução em inglês (sem fonte confiável para inferir a tradução correta).
- Nenhuma técnica foi aplicada "porque existe": cada tratamento acima corresponde a um problema real, confirmado nos dados, e com uma decisão justificada.
