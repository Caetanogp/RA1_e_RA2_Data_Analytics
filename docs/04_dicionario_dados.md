# Dicionário de Dados - Dataset Resultante do RA1

Todos os arquivos estão em `data/processed/`, com a mesma granularidade dos originais em `data/raw/` (um arquivo por tabela; nenhuma junção foi feita - isso é escopo do RA2).

## olist_orders_dataset.csv (99.441 linhas)

| Coluna | Tipo | Descrição | Domínio |
|---|---|---|---|
| order_id | texto | identificador único do pedido | chave primária |
| customer_id | texto | identificador do cliente **para este pedido** (não é a pessoa; ver `olist_customers_dataset.csv`) | chave estrangeira → customers |
| order_status | texto | status atual do pedido | `created`, `approved`, `invoiced`, `processing`, `shipped`, `delivered`, `canceled`, `unavailable` |
| order_purchase_timestamp | datetime | data/hora da compra | sem nulos |
| order_approved_at | datetime | data/hora da aprovação do pagamento | nulo se o pagamento nunca foi aprovado |
| order_delivered_carrier_date | datetime | data/hora de entrega à transportadora | nulo se o pedido não chegou a essa etapa |
| order_delivered_customer_date | datetime | data/hora de entrega ao cliente | nulo se o pedido não foi entregue |
| order_estimated_delivery_date | datetime | data estimada de entrega | sem nulos |

## olist_customers_dataset.csv (99.441 linhas)

| Coluna | Tipo | Descrição | Domínio |
|---|---|---|---|
| customer_id | texto | identificador do cliente **por pedido** | chave primária desta tabela |
| customer_unique_id | texto | identificador real da pessoa, repete-se entre pedidos diferentes do mesmo cliente | - |
| customer_zip_code_prefix | texto (5 dígitos) | prefixo do CEP do cliente | zero-padded |
| customer_city | texto | cidade do cliente | já validado, sem inconsistências |
| customer_state | texto | UF do cliente | uma das 27 UFs brasileiras |

## olist_order_items_dataset.csv (112.650 linhas)

| Coluna | Tipo | Descrição | Domínio |
|---|---|---|---|
| order_id | texto | pedido ao qual o item pertence | chave estrangeira → orders |
| order_item_id | inteiro | número sequencial do item dentro do pedido | ≥ 1 |
| product_id | texto | produto do item | chave estrangeira → products |
| seller_id | texto | vendedor responsável pelo item | chave estrangeira → sellers |
| shipping_limit_date | datetime | prazo limite para envio | sem nulos |
| price | decimal | preço do item (R$) | > 0 |
| freight_value | decimal | valor do frete do item (R$) | ≥ 0 |

## olist_order_payments_dataset.csv (103.883 linhas)

| Coluna | Tipo | Descrição | Domínio |
|---|---|---|---|
| order_id | texto | pedido pago | chave estrangeira → orders |
| payment_sequential | inteiro | ordem da forma de pagamento quando há mais de uma no mesmo pedido | ≥ 1 |
| payment_type | texto | forma de pagamento | `credit_card`, `boleto`, `voucher`, `debit_card` |
| payment_installments | inteiro | número de parcelas | ≥ 1 (corrigido na limpeza; nunca 0) |
| payment_value | decimal | valor pago nesta forma de pagamento (R$) | ≥ 0 (0 é válido para parcelas de voucher fracionado) |

## olist_order_reviews_dataset.csv (99.224 linhas)

| Coluna | Tipo | Descrição | Domínio |
|---|---|---|---|
| review_id | texto | identificador da avaliação - **não é único**, não usar como chave primária isolada | ver nota abaixo |
| order_id | texto | pedido avaliado | chave estrangeira → orders; `(review_id, order_id)` é único |
| review_score | inteiro | nota dada pelo cliente | 1 a 5 |
| review_comment_title | texto | título do comentário (opcional) | pode ser nulo |
| review_comment_message | texto | corpo do comentário (opcional) | pode ser nulo |
| review_creation_date | datetime | data em que a avaliação foi solicitada | sem nulos |
| review_answer_timestamp | datetime | data/hora em que o cliente respondeu | sem nulos |

## olist_products_dataset.csv (32.951 linhas, 10 colunas)

| Coluna | Tipo | Descrição | Domínio |
|---|---|---|---|
| product_id | texto | identificador do produto | chave primária |
| product_category_name | texto | categoria do produto, em português | pode ser nulo (cadastro incompleto) |
| product_category_name_english | texto | tradução da categoria para inglês (via junção) | nulo se categoria ausente ou sem tradução cadastrada |
| product_name_lenght | inteiro | tamanho do nome do produto cadastrado (caracteres) | pode ser nulo |
| product_description_lenght | inteiro | tamanho da descrição cadastrada (caracteres) | pode ser nulo |
| product_photos_qty | inteiro | quantidade de fotos cadastradas | pode ser nulo |
| product_weight_g | decimal | peso do produto (gramas) | > 0 ou nulo (0 foi convertido para nulo na limpeza) |
| product_length_cm, product_height_cm, product_width_cm | decimal | dimensões do produto (cm) | > 0 ou nulo |

## olist_sellers_dataset.csv (3.095 linhas)

| Coluna | Tipo | Descrição | Domínio |
|---|---|---|---|
| seller_id | texto | identificador do vendedor | chave primária |
| seller_zip_code_prefix | texto (5 dígitos) | prefixo do CEP do vendedor | zero-padded |
| seller_city | texto | cidade do vendedor | já validado |
| seller_state | texto | UF do vendedor | uma das 27 UFs brasileiras |

## olist_geolocation_dataset.csv (738.305 linhas)

| Coluna | Tipo | Descrição | Domínio |
|---|---|---|---|
| geolocation_zip_code_prefix | texto (5 dígitos) | prefixo do CEP | zero-padded; não é único (várias coordenadas por prefixo) |
| geolocation_lat | decimal | latitude | entre -34 e 6 (território brasileiro) |
| geolocation_lng | decimal | longitude | entre -74 e -32 (território brasileiro) |
| geolocation_city | texto | cidade associada à coordenada | pode ser nulo; variações de grafia não normalizadas (ver `03_relatorio_qualidade.md`) |
| geolocation_state | texto | UF associada à coordenada | uma das 27 UFs brasileiras |

## product_category_name_translation.csv (71 linhas)

| Coluna | Tipo | Descrição |
|---|---|---|
| product_category_name | texto | categoria em português (chave de junção com `products`) |
| product_category_name_english | texto | tradução em inglês |
