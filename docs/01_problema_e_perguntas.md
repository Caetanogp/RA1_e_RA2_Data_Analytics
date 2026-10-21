# Definição do Problema e Perguntas Analíticas

## Contexto

A Olist é um marketplace brasileiro que conecta pequenos e médios vendedores a grandes marketplaces (como Mercado Livre, Americanas, etc.), centralizando pedidos, produtos, pagamentos e atendimento ao cliente. O dataset público da Olist reúne dados reais e anonimizados de ~99 mil pedidos realizados entre 2016 e 2018, cobrindo todo o ciclo do pedido: compra, pagamento, processamento logístico, entrega e avaliação do cliente.

## Problema

Entender quais fatores operacionais - tempo de entrega, valor de frete, região geográfica, categoria do produto e forma de pagamento - estão associados à satisfação do cliente (nota de avaliação) e à ocorrência de atrasos logísticos nos pedidos da Olist entre 2016 e 2018.

Esse problema é relevante porque, em um marketplace que depende de múltiplos vendedores independentes, a experiência de entrega é um dos principais fatores que o operador da plataforma consegue monitorar e influenciar para manter a satisfação do cliente.

## Perguntas analíticas

As perguntas abaixo orientam as próximas etapas do projeto (ingestão, qualidade, preparação no RA1; transformação e análise nas etapas seguintes do projeto). Elas não são respondidas nesta entrega.

1. Qual a relação entre o atraso na entrega (diferença entre data real e data estimada de entrega) e a nota de avaliação dada pelo cliente?
2. Quais estados/regiões do Brasil apresentam maior incidência de atrasos na entrega?
3. Existe relação entre a categoria do produto e a taxa de atraso ou a nota de avaliação?
4. Como o valor do frete se relaciona com a distância entre vendedor e cliente (a partir dos CEPs/geolocalização) e com a satisfação do cliente?
5. Qual a distribuição de métodos de pagamento e número de parcelas, e como isso se relaciona com o valor total do pedido?

## Fora de escopo neste RA1

Responder às perguntas acima, construir indicadores, agregações de negócio, visualizações exploratórias com fins analíticos e tirar conclusões fazem parte das etapas seguintes do projeto. O RA1 entrega o dataset preparado que servirá de insumo para essa etapa.
