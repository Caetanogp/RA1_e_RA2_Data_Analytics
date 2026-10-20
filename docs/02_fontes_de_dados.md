# Fontes de Dados

## Fonte e origem

**Brazilian E-Commerce Public Dataset by Olist**
- Origem: Kaggle - https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
- Autor: Olist (marketplace brasileiro), publicado pela própria empresa
- Licença: CC BY-NC-SA 4.0 (uso educacional/não comercial)
- Natureza: dados reais e anonimizados de pedidos feitos entre 2016 e 2018, em múltiplos marketplaces brasileiros atendidos pela Olist
- Download reproduzível: o notebook `notebooks/01_ingestao.ipynb` baixa os 9 arquivos originais a partir de um mirror público em GitHub (mesmos arquivos, mesmos nomes e conteúdo do dataset oficial do Kaggle - verificado abaixo), evitando a necessidade de credenciais de API do Kaggle para reproduzir o projeto.

## Formato

9 arquivos CSV relacionais, ligados por chaves (`order_id`, `customer_id`, `product_id`, `seller_id`).

## Atributos e volumetria real (obtida na ingestão)

| Arquivo | Linhas | Colunas | Principais atributos |
|---|---|---|---|
| `olist_orders_dataset.csv` | 99.441 | 8 | `order_id`, `customer_id`, `order_status`, `order_purchase_timestamp`, `order_approved_at`, `order_delivered_carrier_date`, `order_delivered_customer_date`, `order_estimated_delivery_date` |
| `olist_customers_dataset.csv` | 99.441 | 5 | `customer_id`, `customer_unique_id`, `customer_zip_code_prefix`, `customer_city`, `customer_state` |
| `olist_order_items_dataset.csv` | 112.650 | 7 | `order_id`, `order_item_id`, `product_id`, `seller_id`, `shipping_limit_date`, `price`, `freight_value` |
| `olist_order_payments_dataset.csv` | 103.886 | 5 | `order_id`, `payment_sequential`, `payment_type`, `payment_installments`, `payment_value` |
| `olist_order_reviews_dataset.csv` | 99.224 | 7 | `review_id`, `order_id`, `review_score`, `review_comment_title`, `review_comment_message`, `review_creation_date`, `review_answer_timestamp` |
| `olist_products_dataset.csv` | 32.951 | 9 | `product_id`, `product_category_name`, dimensões (`_lenght`, `_weight_g`, `_length_cm`, `_height_cm`, `_width_cm`), `product_photos_qty` |
| `olist_sellers_dataset.csv` | 3.095 | 4 | `seller_id`, `seller_zip_code_prefix`, `seller_city`, `seller_state` |
| `olist_geolocation_dataset.csv` | 1.000.163 | 5 | `geolocation_zip_code_prefix`, `geolocation_lat`, `geolocation_lng`, `geolocation_city`, `geolocation_state` |
| `product_category_name_translation.csv` | 71 | 2 | `product_category_name`, `product_category_name_english` |

## Relação com o problema

Este conjunto cobre todo o ciclo de um pedido no marketplace - compra, pagamento, item/produto, vendedor, logística de entrega e avaliação do cliente - permitindo relacionar desempenho de entrega e características do pedido (frete, categoria, região) com a satisfação do cliente (nota de avaliação), que é exatamente o problema definido em `docs/01_problema_e_perguntas.md`.
