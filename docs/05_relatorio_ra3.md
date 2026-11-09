# Relatório do RA3 - Análise estatística, visualizações e Machine Learning

Implementação: `notebooks/05_analise_ra3.ipynb` (executado, com saídas e gráficos gravados). Os dados vêm de `data/processed/`, preparados no RA1. Este documento resume as técnicas, os resultados e as interpretações; o detalhe e os gráficos estão no notebook.

## Base de análise

Os 9 arquivos foram juntados em **uma linha por pedido**, usando só os pedidos entregues que têm itens e pagamento registrados: **96.469 pedidos** (97,0% dos 99.441 da base). Atraso é a data de entrega menos a data estimada; um pedido é "atrasado" quando esse valor é maior que zero.

## 1. Análise estatística descritiva

| Medida | Para quê | Onde |
|---|---|---|
| Média e mediana | valor típico de tempo de entrega, atraso, preço, frete e valor pago | tabela da seção 1 |
| Desvio-padrão, amplitude e quartis | variabilidade | tabela da seção 1 |
| Assimetria, frequência da nota, proporção de atrasados | forma da distribuição | seção 1 |

Resultados principais: a mediana do tempo de entrega é 10 dias (média 12,1; máximo 209); o pedido típico chega 12 dias antes da data estimada; só 6,77% dos pedidos atrasam; preço e frete têm média bem acima da mediana (poucos pedidos muito caros), por isso a mediana é a medida de referência; 57,8% das notas são 5 e 11,5% são 1.

## 2. Visualizações e interpretação

| Gráfico | Pergunta | Resultado |
|---|---|---|
| Histograma do tempo de entrega | distribuição | metade dos pedidos chega entre 6 e 15 dias, com cauda longa |
| Nota média por faixa de atraso | 1 | 4,29 no prazo, 3,29 com 1 a 3 dias, 2,11 com 4 a 7 dias, 1,70 com mais de 7 dias |
| Taxa de atraso por estado | 2 | de 2,8% (AM) a 21,4% (AL); os cinco maiores são do Nordeste |
| Taxa de atraso por categoria | 3 | 5,4% a 7,5% nas 10 categorias mais vendidas: diferença pequena |
| Frete por mesmo estado ou não | 4 | frete mediano R$ 12,70 contra R$ 19,04; atraso 4,5% contra 8,0% |
| Valor do pedido por parcelas | 5 | mediana de R$ 79,24 à vista até R$ 206,12 em 7 a 10 parcelas |

## 3. Amostragem e representatividade

Não houve amostragem: a análise usa todos os pedidos entregues. Limites: um só marketplace e período de 2016 a 2018 (2016 com 266 pedidos), 42,0% dos pedidos em SP, nota voluntária (69,3% das avaliações são 1 ou 5), pedidos cancelados e indisponíveis fora da análise e 1.351 pedidos sem categoria.

## 4. Machine Learning

- **Tipo:** classificação. **Problema:** prever se um pedido será entregue com atraso. **Target:** `atrasado`.
- **Características:** estado do cliente, estado do vendedor, mesmo estado, categoria (10 mais frequentes e "outras"), preço, frete, peso, número de itens, prazo estimado, parcelas, tipo de pagamento e mês da compra. Tempo de entrega, dias de atraso e nota ficam de fora, porque só existem depois da entrega (evita vazamento de dados).
- **Divisão:** 80% treino (77.175 pedidos) e 20% teste (19.294), estratificada, com `random_state=42`.
- **Modelos:** referência que sempre responde "no prazo" (`DummyClassifier`) e `RandomForestClassifier` com `class_weight="balanced"`.

| Modelo | Acurácia | Precisão | Recall | F1 | AUC |
|---|---|---|---|---|---|
| Sempre "no prazo" | 0,932 | 0,000 | 0,000 | 0,000 | 0,500 |
| Floresta aleatória | 0,741 | 0,155 | 0,634 | 0,249 | 0,757 |

A acurácia engana, porque 93,2% dos pedidos são pontuais. A floresta encontra 63,4% dos atrasados (828 de 1.307 no teste), mas com muitos alarmes falsos (precisão de 15,5%): serve para chamar atenção a pedidos de risco, não para decidir sozinha. A característica mais importante é o mês da compra, e os atrasos se concentram em março de 2018 (19,0%), fevereiro de 2018 (14,1%) e novembro de 2017 (12,4%).

## 5. Conclusões

- O atraso é o fator mais claramente ligado à nota; região e distância aproximada se ligam ao atraso; o período da compra pesa muito; a categoria do produto pesa pouco.
- O modelo identifica parte dos pedidos de risco já na compra, com desempenho modesto.
- As relações são associações, não causas. Limitações: um marketplace, 2016 a 2018, distância aproximada por "mesmo estado" e nenhuma informação de transportadora ou eventos externos.

## Como executar

O notebook só lê `data/processed/` (já no repositório) e leva cerca de 1 minuto.

```bash
pip install -r requirements.txt
jupyter notebook notebooks/
```

Abra `05_analise_ra3.ipynb` e use *Restart & Run All*. Resultado esperado: seis gráficos, a tabela de métricas (floresta aleatória: recall 0,634 e AUC 0,757) e a seção de conclusões ao final. Como as sementes são fixas, os números se repetem.

Com Docker (Docker Desktop aberto), na raiz do repositório:

```powershell
docker build -f docker/Dockerfile -t ra2-spark .
docker run --rm -v "${PWD}:/work" -w /work/notebooks ra2-spark jupyter nbconvert --to notebook --execute --inplace 05_analise_ra3.ipynb
```
