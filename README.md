# Olist - Ingestão, qualidade e streaming de dados (RA1 e RA2)

Projeto de dados sobre o **Brazilian E-Commerce Public Dataset by Olist**: ~99 mil pedidos reais e anonimizados de um marketplace brasileiro (2016-2018). O repositório reúne duas etapas:

- **RA1 - Ingestão e preparação dos dados:** baixa os dados por código, avalia a qualidade e entrega um conjunto limpo e documentado.
- **RA2 - Processamento em streaming:** simula a chegada de pedidos e calcula a taxa de atraso de entrega por janela de tempo com Apache Spark Structured Streaming.

**Equipe:** Caetano Padoin, Davi Henrique e Matheus Brehm - Data Analytics (PjBL).

**Fonte dos dados:** [Kaggle - Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), licença CC BY-NC-SA 4.0 (uso educacional).

## Estrutura

```
.
├── README.md
├── requirements.txt
├── docker/Dockerfile      # ambiente com Spark para rodar o RA2
├── data/
│   ├── raw/               # CSVs originais (baixados pelo notebook 01; não versionados)
│   └── processed/         # conjunto resultante do RA1 (9 CSVs, gerados pelo notebook 03)
├── notebooks/
│   ├── 01_ingestao.ipynb
│   ├── 02_avaliacao_qualidade.ipynb
│   ├── 03_limpeza_preparacao.ipynb
│   ├── 04_streaming_ra2.ipynb
│   ├── entrega_ra1_ra2.ipynb   # RA1 + RA2 em um único notebook
│   └── entrega_ra1_ra2.pdf
└── docs/                  # documentação do projeto (ver tabela abaixo)
```

## Como rodar

### Pré-requisitos

- Python 3.12 ou superior.
- Para o RA2: Docker (Windows) ou Java 17+ (Linux/macOS), conforme abaixo.

```bash
pip install -r requirements.txt
```

### RA1 - notebooks 01 a 03

Abra a pasta de notebooks e execute **em ordem**, cada um com *Restart & Run All*:

```bash
jupyter notebook notebooks/
```

| Notebook | O que faz | Como saber que deu certo |
|---|---|---|
| `01_ingestao.ipynb` | Baixa os 9 CSVs originais de um mirror público (precisa de internet, ~130 MB) e salva em `data/raw/` | A última célula mostra a contagem de linhas de cada arquivo, por exemplo 99.441 pedidos e 1.000.163 pontos de geolocalização |
| `02_avaliacao_qualidade.ipynb` | Investiga valores ausentes, duplicados, tipos, valores inválidos, categorias e outliers | Termina com a tabela "Resumo dos achados" |
| `03_limpeza_preparacao.ipynb` | Trata os problemas encontrados e grava os 9 CSVs em `data/processed/` | Imprime "Checagens de valor inválido/outlier passaram." e o resumo dos 9 arquivos |

`data/raw/` não é versionado e nunca é editado à mão: o notebook 01 o recria. O conjunto resultante, `data/processed/`, já está no repositório; para rodar só o RA2, não é preciso refazer o RA1.

### RA2 - streaming com Spark

O notebook `notebooks/04_streaming_ra2.ipynb` grava 3 lotes com 15 pedidos simulados (escritos à mão) numa pasta de entrada, em momentos diferentes, e o Spark os processa conforme chegam, agrupando por **janela tumbling de 10 minutos**. Ele não depende dos dados do RA1.

**No Windows** o Spark exige binários do Hadoop (`winutils`). O jeito mais simples é usar o container deste repositório (Docker Desktop aberto):

```powershell
docker build -f docker/Dockerfile -t ra2-spark .
docker run --rm -v "${PWD}:/work" -w /work/notebooks ra2-spark jupyter nbconvert --to notebook --execute --inplace 04_streaming_ra2.ipynb
```

No Linux/macOS (bash), troque `${PWD}` por `$PWD`. Sem Docker, basta ter Java 17+ e rodar o notebook normalmente com `jupyter notebook notebooks/`.

**Como saber que deu certo:** o notebook mostra, a cada lote, a tabela das janelas e termina com `Total de pedidos nas janelas: 15 (de 15 enviados)`.

O notebook `entrega_ra1_ra2.ipynb` junta o RA1 e o RA2 em um único arquivo, no formato pedido pela disciplina, e `entrega_ra1_ra2.pdf` é a sua versão em PDF. Ele é gerado a partir dos demais notebooks.

## Documentação

| Documento | Conteúdo |
|---|---|
| [`docs/01_problema_e_perguntas.md`](docs/01_problema_e_perguntas.md) | Problema analítico e perguntas norteadoras |
| [`docs/02_fontes_de_dados.md`](docs/02_fontes_de_dados.md) | Origem, formato e atributos de cada arquivo |
| [`docs/03_relatorio_qualidade.md`](docs/03_relatorio_qualidade.md) | Problemas de qualidade encontrados e decisões de tratamento |
| [`docs/04_dicionario_dados.md`](docs/04_dicionario_dados.md) | Dicionário de dados do conjunto resultante |
| [`docs/RA1_documentacao_preenchida.docx`](docs/RA1_documentacao_preenchida.docx) | Documentação completa do RA1 (formato Word da disciplina) |

## Escopo

O RA1 cobre apenas até a preparação dos dados: sem transformações analíticas, indicadores ou conclusões de negócio. O RA2 é uma etapa à parte, de streaming sobre registros **simulados**, sem dashboards nem Kafka; as taxas que ele mostra não representam a operação real da Olist.
