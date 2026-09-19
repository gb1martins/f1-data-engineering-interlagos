### Projeto Engenharia de Dados aplicada à Fórmula 1: Interlagos  

## Objetivo

A partir de dados históricos da Fórmula 1, o projeto tem como objetivo construir um pipeline de Engenharia de Dados capaz de coletar, armazenar, transformar, integrar e disponibilizar informações relacionadas ao Grande Prêmio de São Paulo, realizado no Autódromo de Interlagos.

Sobre essa base de dados, buscamos identificar quais características de desempenho, estratégia e contexto estão associadas às vitórias em Interlagos e utilizar esse histórico como referência para analisar o desempenho de Gabriel Bortoleto.

O objetivo não é estabelecer uma regra determinística para vencer uma corrida, mas compreender os padrões observados historicamente entre os pilotos vencedores e comparar esses padrões com as características apresentadas por Gabriel Bortoleto em suas corridas em Interlagos.

---

## Contexto e problema

A Fórmula 1 é uma competição caracterizada por elevada complexidade, na qual o resultado de uma corrida é influenciado pela combinação de diferentes fatores, como posição de largada, ritmo de corrida, estratégia de pit stops, utilização dos pneus, condições climáticas e eventos ocorridos durante a prova.

No Grande Prêmio de São Paulo, realizado em Interlagos, esses fatores podem variar significativamente entre temporadas e corridas. Dessa forma, analisar apenas o resultado final ou uma variável isolada não é suficiente para compreender os padrões associados às vitórias.

Nesse contexto, surge a necessidade de integrar diferentes fontes de dados de Fórmula 1, com diferentes estruturas e granularidades, transformando dados brutos em informações confiáveis e organizadas para análise.

O desafio central do projeto consiste, portanto, em construir uma solução de Engenharia de Dados capaz de realizar todo esse fluxo — desde a ingestão dos dados até sua disponibilização para análise — e, a partir dos dados consolidados, identificar os principais fatores historicamente associados às vitórias em Interlagos.

Como aplicação desse estudo, esses fatores são utilizados para realizar uma análise comparativa do desempenho de Gabriel Bortoleto em relação ao histórico de pilotos vencedores no circuito.

A etapa de Machine Learning complementa essa análise ao permitir avaliar associações entre as características das corridas e a ocorrência de vitória, além de contribuir para a interpretação dos fatores mais relevantes no modelo.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Coleta dos dados

1.Dataset bruto(raw data)

Foram coletados dados reais de Fórmula 1 em formato JSON, armazenados na camada Bronze do MinIO.

As bases utilizadas são:

Resultados das corridas: 2015–2025
Pit Stops: 2015–2025
Voltas: 2015–2025
Calendário: 2015–2025
Clima: 2018–2025
Pneus: 2018–2025


2.Lista/documentação das fonte dos dados

Fontes e requisições

| Dataset    | Fonte          | Requisição / Endpoint                                            | Período   |
| ---------- | -------------- | ---------------------------------------------------------------- | --------- |
| Resultados | Jolpica F1 API | `/ergast/f1/{ano}/{round}/results/`                              | 2015–2025 |
| Pit Stops  | Jolpica F1 API | `/ergast/f1/{ano}/{round}/pitstops/`                             | 2015–2025 |
| Voltas     | Jolpica F1 API | `/ergast/f1/{ano}/{round}/laps/`                                 | 2015–2025 |
| Calendário | Jolpica F1 API | `/ergast/f1/{ano}.json`                                          | 2015–2025 |
| Clima      | FastF1         | `get_session(ano, "São Paulo", sessão)` → `session.weather_data` | 2018–2025 |
| Pneus      | FastF1         | `get_session(ano, "São Paulo", sessão)` → `session.laps`         | 2018–2025 |

### Jolpica F1 API

Os dados de resultados, pit stops, voltas e calendário são coletados
por meio de requisições HTTP à Jolpica F1 API, utilizando os endpoints
correspondentes a cada dataset.

### FastF1

Os dados de clima e pneus são obtidos por meio da biblioteca FastF1.
A sessão do Grande Prêmio de São Paulo é carregada e os dados de
`weather_data` e `laps` são extraídos conforme o dataset.


3.Dicionário de dados (data dictionary)

Resultados

| Campo                     | Tipo    | Descrição                                        |
| ------------------------- | ------- | ------------------------------------------------ |
| `season`                  | INTEGER | Temporada da Fórmula 1                           |
| `round`                   | INTEGER | Número da etapa no campeonato                    |
| `race_name`               | VARCHAR | Nome do Grande Prêmio                            |
| `race_date`               | DATE    | Data da corrida                                  |
| `race_time`               | VARCHAR | Horário programado da corrida                    |
| `race_time_millis`        | BIGINT  | Horário da corrida convertido para milissegundos |
| `race_url`                | VARCHAR | URL de referência da corrida                     |
| `circuit_id`              | VARCHAR | Identificador do circuito                        |
| `circuit_name`            | VARCHAR | Nome do circuito                                 |
| `circuit_url`             | VARCHAR | URL de referência do circuito                    |
| `circuit_lat`             | DOUBLE  | Latitude do circuito                             |
| `circuit_long`            | DOUBLE  | Longitude do circuito                            |
| `circuit_locality`        | VARCHAR | Cidade/localidade do circuito                    |
| `circuit_country`         | VARCHAR | País do circuito                                 |
| `driver_id`               | VARCHAR | Identificador do piloto                          |
| `driver_number`           | INTEGER | Número do piloto                                 |
| `driver_code`             | VARCHAR | Código de três letras do piloto                  |
| `constructor_id`          | VARCHAR | Identificador da equipe                          |
| `constructor_name`        | VARCHAR | Nome da equipe                                   |
| `position`                | INTEGER | Posição final do piloto                          |
| `points`                  | DOUBLE  | Pontos obtidos na corrida                        |
| `grid`                    | INTEGER | Posição de largada                               |
| `laps`                    | INTEGER | Número de voltas completadas                     |
| `status`                  | VARCHAR | Status final do piloto na corrida                |
| `race_time_millis`        | BIGINT  | Tempo de corrida convertido para milissegundos   |
| `fastest_lap`             | INTEGER | Número da volta mais rápida                      |
| `fastest_lap_time`        | VARCHAR | Tempo da volta mais rápida                       |
| `fastest_lap_time_millis` | BIGINT  | Tempo da volta mais rápida em milissegundos      |
| `fastest_lap_rank`        | INTEGER | Classificação da volta mais rápida               |
| `average_speed`           | DOUBLE  | Velocidade média da volta mais rápida            |
| `source_url`              | VARCHAR | URL da fonte dos dados                           |


Pit Stops

| Campo              | Tipo    | Descrição                             |
| ------------------ | ------- | ------------------------------------- |
| `season`           | INTEGER | Temporada                             |
| `round`            | INTEGER | Número da etapa                       |
| `race_name`        | VARCHAR | Nome do Grande Prêmio                 |
| `race_date`        | DATE    | Data da corrida                       |
| `race_time`        | VARCHAR | Horário da corrida                    |
| `race_time_millis` | BIGINT  | Horário convertido para milissegundos |
| `race_url`         | VARCHAR | URL da corrida                        |
| `circuit_id`       | VARCHAR | Identificador do circuito             |
| `circuit_name`     | VARCHAR | Nome do circuito                      |
| `circuit_url`      | VARCHAR | URL do circuito                       |
| `circuit_lat`      | DOUBLE  | Latitude                              |
| `circuit_long`     | DOUBLE  | Longitude                             |
| `circuit_locality` | VARCHAR | Localidade                            |
| `circuit_country`  | VARCHAR | País                                  |
| `driver_id`        | VARCHAR | Identificador do piloto               |
| `lap`              | INTEGER | Volta em que ocorreu o pit stop       |
| `stop`             | INTEGER | Número do pit stop do piloto          |
| `pit_stop_time`    | TIME    | Horário em que ocorreu a parada       |
| `duration`         | DOUBLE  | Duração da parada em segundos         |
| `duration_millis`  | BIGINT  | Duração da parada em milissegundos    |

Calendário

| Campo                  | Tipo    | Descrição                             |
| ---------------------- | ------- | ------------------------------------- |
| `season`               | INTEGER | Temporada                             |
| `round`                | INTEGER | Número da etapa                       |
| `race_name`            | VARCHAR | Nome do Grande Prêmio                 |
| `race_date`            | DATE    | Data da corrida                       |
| `race_time`            | VARCHAR | Horário da corrida                    |
| `race_time_millis`     | BIGINT  | Horário convertido para milissegundos |
| `race_url`             | VARCHAR | URL da corrida                        |
| `circuit_id`           | VARCHAR | Identificador do circuito             |
| `circuit_name`         | VARCHAR | Nome do circuito                      |
| `circuit_url`          | VARCHAR | URL do circuito                       |
| `circuit_lat`          | DOUBLE  | Latitude                              |
| `circuit_long`         | DOUBLE  | Longitude                             |
| `circuit_locality`     | VARCHAR | Localidade                            |
| `circuit_country`      | VARCHAR | País                                  |
| `first_practice_date`  | DATE    | Data do primeiro treino               |
| `second_practice_date` | DATE    | Data do segundo treino                |
| `third_practice_date`  | DATE    | Data do terceiro treino               |
| `qualifying_date`      | DATE    | Data da classificação                 |
| `source_url`           | VARCHAR | URL da fonte                          |

Voltas

| Campo              | Tipo    | Descrição                       |
| ------------------ | ------- | ------------------------------- |
| `season`           | INTEGER | Temporada                       |
| `round`            | INTEGER | Número da etapa                 |
| `race_name`        | VARCHAR | Nome do Grande Prêmio           |
| `race_date`        | DATE    | Data da corrida                 |
| `race_time`        | VARCHAR | Horário da corrida              |
| `race_time_millis` | BIGINT  | Horário em milissegundos        |
| `total_laps`       | INTEGER | Total de voltas da corrida      |
| `circuit_id`       | VARCHAR | Identificador do circuito       |
| `circuit_name`     | VARCHAR | Nome do circuito                |
| `circuit_url`      | VARCHAR | URL do circuito                 |
| `circuit_lat`      | DOUBLE  | Latitude                        |
| `circuit_long`     | DOUBLE  | Longitude                       |
| `circuit_locality` | VARCHAR | Localidade                      |
| `circuit_country`  | VARCHAR | País                            |
| `lap`              | INTEGER | Número da volta                 |
| `driver_id`        | VARCHAR | Identificador do piloto         |
| `lap_time`         | VARCHAR | Tempo original da volta         |
| `position`         | INTEGER | Posição do piloto na volta      |
| `lap_time_seconds` | DOUBLE  | Tempo da volta em segundos      |
| `lap_time_millis`  | BIGINT  | Tempo da volta em milissegundos |

Clima

| Campo                  | Tipo    | Descrição                         |
| ---------------------- | ------- | --------------------------------- |
| `season`               | INTEGER | Temporada                         |
| `grand_prix`           | VARCHAR | Grande Prêmio                     |
| `circuit`              | VARCHAR | Circuito                          |
| `session`              | VARCHAR | Identificador da sessão           |
| `session_name`         | VARCHAR | Nome da sessão                    |
| `event_name`           | VARCHAR | Nome do evento                    |
| `location`             | VARCHAR | Localização                       |
| `event_date`           | DATE    | Data do evento                    |
| `weather_time_seconds` | DOUBLE  | Tempo da medição em segundos      |
| `weather_time_millis`  | BIGINT  | Tempo da medição em milissegundos |
| `air_temp`             | DOUBLE  | Temperatura do ar                 |
| `humidity`             | DOUBLE  | Umidade                           |
| `pressure`             | DOUBLE  | Pressão atmosférica               |
| `rainfall`             | BOOLEAN | Indicador de chuva                |
| `track_temp`           | DOUBLE  | Temperatura da pista              |
| `wind_direction`       | INTEGER | Direção do vento                  |
| `wind_speed`           | DOUBLE  | Velocidade do vento               |

Pneus

| Campo           | Tipo    | Descrição                 |
| --------------- | ------- | ------------------------- |
| `season`        | INTEGER | Temporada                 |
| `grand_prix`    | VARCHAR | Grande Prêmio             |
| `circuit`       | VARCHAR | Circuito                  |
| `session`       | VARCHAR | Sessão                    |
| `session_name`  | VARCHAR | Nome da sessão            |
| `event_name`    | VARCHAR | Nome do evento            |
| `location`      | VARCHAR | Localização               |
| `event_date`    | DATE    | Data do evento            |
| `driver_id`     | VARCHAR | Identificador do piloto   |
| `driver_number` | INTEGER | Número do piloto          |
| `lap_number`    | INTEGER | Número da volta           |
| `stint`         | INTEGER | Número do stint           |
| `compound`      | VARCHAR | Composto do pneu          |
| `tyre_life`     | INTEGER | Idade do pneu em voltas   |
| `fresh_tyre`    | BOOLEAN | Indica se o pneu era novo |

Scripts e processos de coleta

A coleta dos dados é realizada por scripts Python responsáveis por consultar as fontes de dados, processar as respostas e armazená-las inicialmente na camada Bronze do Data Lake.

O processo segue o fluxo:

Fonte de dados → Python → MinIO (Bronze)

Os scripts realizam as seguintes etapas:

Definição do período e dos parâmetros de coleta;
Consulta às fontes de dados;
Recebimento dos dados em formato JSON;
Validação da resposta da fonte;
Organização dos dados coletados;
Armazenamento do JSON bruto no MinIO, preservando os dados originais para posterior processamento.

Para os dados provenientes da Jolpica F1 API, os scripts realizam requisições HTTP aos endpoints correspondentes a cada dataset.

Para os dados de clima e pneus, é utilizada a biblioteca FastF1, que permite carregar as sessões dos eventos e extrair os dados de weather_data e laps.

Tecnologias utilizadas
Python — execução dos scripts de ingestão;
Requests — requisições HTTP à API;
FastF1 — coleta dos dados de sessões, clima e pneus;
Boto3 — comunicação com o armazenamento S3-compatible;
MinIO — armazenamento dos dados na camada Bronze;
JSON — formato dos dados brutos.
Organização da camada Bronze

Os dados são armazenados no MinIO de forma organizada por dataset e temporada, permitindo que os arquivos brutos sejam posteriormente utilizados pelos processos de transformação da camada Silver.

Princípio utilizado: a camada Bronze mantém os dados coletados o mais próximo possível do formato disponibilizado pela fonte, enquanto as conversões de tipos, padronizações e demais tratamentos são realizados posteriormente na camada Silver.


Critérios de seleção dos dados

Foram selecionados dados relevantes para análise do desempenho nas corridas de Fórmula 1, priorizando:

Fontes públicas e confiáveis, como a Jolpica F1 API e a biblioteca FastF1;
Qualidade e estrutura dos dados;
Possibilidade de integração entre os datasets;
Relevância para o projeto;
Período histórico disponível.

Os datasets selecionados abrangem resultados, voltas, pit stops, calendário, clima e pneus.


### Pré-processamento

Abordagens utilizadas:

Arquitetura Medallion: dados brutos mantidos na camada Bronze e tratados na camada Silver.
ELT: os dados são primeiro armazenados no MinIO e posteriormente transformados utilizando DuckDB.
Padronização e tipagem: conversão de tipos, TRIM() dos campos textuais e criação de campos derivados, como tempos em segundos e milissegundos.
Validação de qualidade: verificação de duplicidades utilizando chaves lógicas específicas para cada dataset.
PyArrow: utilizado para registrar os dados no DuckDB e gerar as tabelas processadas em parquet.

Justificativa:

A abordagem foi escolhida para preservar os dados originais na Bronze, permitindo rastreabilidade e reprocessamento, enquanto a Silver concentra a padronização, tipagem e validação dos dados para facilitar sua integração e utilização nas análises.


---

## Análise Exploratória de Dados

A EDA histórica é realizada sobre os dados disponibilizados na camada Silver e
tem como objetivo compreender a qualidade dos dados, suas distribuições,
relações entre variáveis e principais padrões observados nas corridas.

Entre os temas analisados estão:

- qualidade e integridade dos dados;
- resultados das corridas;
- posição de largada e posição final;
- ritmo de corrida;
- pit stops;
- pneus e stints;
- condições climáticas;
- análises combinadas;
- análises estatísticas;
- principais insights e conclusões.

A EDA histórica não representa uma camada da arquitetura. Ela é uma etapa
analítica que utiliza os dados disponibilizados pela Silver.

### Streamlit

As análises e visualizações são disponibilizadas por meio de uma aplicação interativa em Streamlit, permitindo explorar os dados consolidados e os principais resultados do projeto.

- [Código da aplicação Streamlit](streamlit/app.py)

A aplicação funciona como camada de apresentação e exploração dos dados e não substitui as etapas de processamento do pipeline.

### Modelagem

Após a disponibilização dos dados na Silver, é realizada a modelagem da camada Gold, responsável por organizar os dados em estruturas analíticas de dimensões e fatos.

A **modelagem, granularidades, relacionamentos, regras de negócio e validações da Gold** estão documentadas separadamente:

- [Documentação da Modelagem Gold](gold/docs/MODELAGEM_GOLD.md)


### Machine Learning e análise dos fatores associados à vitória

A partir dos dados consolidados na Gold, é construído o dataset utilizado na etapa de Machine Learning.

A etapa de ML complementa o pipeline de Engenharia de Dados com a construção e validação do dataset, EDA específica, preparação das features, separação entre desenvolvimento e teste, treinamento, avaliação, tuning e interpretação dos modelos.

A documentação completa da etapa está disponível nos arquivos abaixo:

- [Documentação principal — Etapa de Machine Learning](ml/docs/DOC_ML_INTERLAGOS_REESTRUTURADO.docx)
- [Documentação técnica complementar de ML](ml/docs/DOCUMENTACAO_TECNICA_COMPLEMENTAR_ML_INTERLAGOS.md)

## Principais resultados

A partir dos dados consolidados nas camadas analíticas e da etapa de Machine Learning, foi possível investigar quais características estão associadas às vitórias em Interlagos e utilizar esse histórico como referência para a análise comparativa de Gabriel Bortoleto.

### Resultado do modelo

O processo de Machine Learning utilizou o **F1-score como métrica principal para o tuning**, devido ao forte desbalanceamento entre vitórias e não vitórias. O **ROC-AUC** foi utilizado como métrica complementar.

O **Random Forest** apresentou o melhor resultado médio de F1-score durante o tuning e foi utilizado como modelo final.

No conjunto de teste, composto pelas temporadas de 2024 e 2025:

| Métrica | Resultado |
|---|---:|
| Accuracy | 95,00% |
| Precision | 0,00% |
| Recall | 0,00% |
| F1-score | 0,00% |
| ROC-AUC | 1,00 |

Apesar do ROC-AUC elevado, o modelo não classificou nenhuma observação como vitória utilizando o limiar de 0,5. Entretanto, o piloto vencedor foi ranqueado em primeiro lugar pela probabilidade estimada nas duas corridas do conjunto de teste.

Os resultados devem ser interpretados com cautela devido ao pequeno número de corridas disponíveis no conjunto de teste.

### Importância das variáveis

A análise de importância das variáveis do Random Forest indicou maior relevância para o **ritmo de corrida** e a **posição de largada**, seguidos por características relacionadas às paradas nos boxes, cobertura do ritmo e estratégia de pneus.

![Importância das variáveis — Random Forest](ml/visualization/outputs/04_importancia_features.png)


### Importância por grupo

Para facilitar a interpretação, as variáveis foram agrupadas em categorias analíticas:

| Rank | Grupo | Importância |
|---:|---|---:|
| 1 | Desempenho | 30,38% |
| 2 | Qualificação | 19,39% |
| 3 | Cobertura do Ritmo | 16,35% |
| 4 | Pit Stops | 16,31% |
| 5 | Stints e Pneus | 13,37% |
| 6 | Clima | 4,20% |

Entre os grupos analisados, **Desempenho** apresentou a maior importância relativa (30,38%), seguido por **Qualificação** (19,39%). Os grupos relacionados à estratégia de corrida, como **Pit Stops** (16,31%) e **Stints e Pneus** (13,37%), também apresentaram participação relevante.

> A importância das variáveis representa a contribuição relativa utilizada pelo modelo para discriminar as classes no conjunto de desenvolvimento. Esses valores não devem ser interpretados como efeitos causais nem como uma regra determinística para vencer uma corrida.

### Aplicação ao caso de Gabriel Bortoleto

Os resultados foram utilizados como referência para comparar as características apresentadas por **Gabriel Bortoleto** em Interlagos com o histórico observado entre os pilotos vencedores.

As análises comparativas consideram principalmente:

- ritmo de corrida;
- posição de largada;
- estratégia de pit stops;
- stints e utilização dos pneus;
- condições climáticas;
- comparação com o perfil histórico de vencedores.

A análise possui caráter **descritivo e retrospectivo**, buscando identificar proximidades e diferenças entre o desempenho de Gabriel e os padrões históricos observados em Interlagos.

---

## Apresentação do projeto

A apresentação com a visão geral do projeto, arquitetura, metodologia, resultados e principais conclusões está disponível na pasta de apresentação do repositório:

- [Apresentação do Projeto — PowerPoint](ppt/Projeto_Engenharia_De_Dados_Formula_1_Interlagos.pptx)


---

## Como executar

O projeto atualmente depende de execução sequencial dos scripts, pois as etapas ainda não estão orquestradas por uma ferramenta de workflow. A ordem abaixo deve ser respeitada para garantir que cada camada tenha sido construída e validada antes de alimentar a etapa seguinte.

### 1. Clonar o repositório

```bash
git clone https://github.com/gb1martins/f1-data-engineering-interlagos.git

cd f1-data-engineering-interlagos
```

### 2. Criar o ambiente virtual

```powershell
python -m venv .venv
.venv\Scripts\activate
```

### 3. Instalar as dependências

```powershell
pip install -r requirements.txt
```

### 4. Subir a infraestrutura local

```powershell
docker compose up -d
docker ps
```

O Docker disponibiliza a infraestrutura utilizada pelo projeto, incluindo o MinIO para armazenamento dos dados no Data Lake.

### 5. Executar a ingestão — Bronze

Os scripts de ingestão coletam os dados das fontes e armazenam os arquivos brutos no MinIO.

```powershell
python ingestao/jolpica/ingerir_calendario.py
python ingestao/jolpica/ingerir_resultado.py
python ingestao/jolpica/ingerir_voltas.py
python ingestao/jolpica/ingerir_pit_stops.py
python ingestao/fastf1/ingerir_clima.py
python ingestao/fastf1/ingerir_pneus.py
```

Fluxo:

```text
Fontes → Python → MinIO (Bronze)
```

### 6. Executar o pré-processamento — Silver

Após a ingestão, os dados são padronizados, tipados, validados e consolidados na camada Silver.

```powershell
python pre_processamento/silver_calendario.py
python pre_processamento/silver_resultados.py
python pre_processamento/silver_voltas.py
python pre_processamento/silver_pit_stops.py
python pre_processamento/silver_clima.py
python pre_processamento/silver_pneus.py
python pre_processamento/silver_consolidar_arquivos.py
python pre_processamento/silver_driver_mapping.py
```

Ao final dessa etapa, os dados tratados estarão disponíveis na camada Silver.

### 7. Explorar os dados no Streamlit

A aplicação utiliza os dados disponibilizados na Silver para apresentar as análises exploratórias e visualizações.

Antes da execução, configure as credenciais do MinIO no arquivo:

```text
.streamlit/secrets.toml
```

Exemplo:

```toml
[minio]
endpoint = "localhost:9000"
access_key = "admin"
secret_key = "minioadmin123"
use_ssl = false
```

Execute:

```powershell
streamlit run streamlit/app.py
```

> A aplicação Streamlit é uma camada de apresentação e exploração. Ela não substitui as etapas de processamento do pipeline.

### 8. Construir a camada Gold

Após a conclusão da Silver:

```powershell
python gold/scripts/build_gold.py
```

A documentação detalhada está em:

```text
gold/docs/MODELAGEM_GOLD.md
```

### 9. Validar a camada Gold

```powershell
python gold/scripts/validate_gold.py
pytest gold/tests -v
```

### 10. Construir o dataset de Machine Learning

```powershell
python ml/feature_engineering/build_dataset_ml.py
```

### 11. Validar o dataset de Machine Learning

```powershell
python ml/feature_engineering/valida_dataset_ml.py
```

### 12. Executar a EDA específica de ML

```powershell
python ml/feature_engineering/eda_dataset_ml.py
```

A EDA avalia a distribuição do target, associações entre features, valores ausentes e redundâncias entre variáveis.

### 13. Preparar os dados para modelagem

```powershell
python ml/feature_engineering/prepare_model_data.py
```

### 14. Separar desenvolvimento e teste

A separação é realizada por corrida (`race_key`), mantendo as corridas do conjunto de teste isoladas das corridas utilizadas no desenvolvimento.

```powershell
python ml/modeling/split_data.py
```

### 15. Treinar os modelos iniciais

```powershell
python ml/modeling/train_models.py
```

### 16. Avaliar os modelos iniciais

```powershell
python ml/modeling/evaluate_initial.py
```

### 17. Realizar o tuning dos modelos

```powershell
python ml/modeling/tune_models.py
```

O tuning utiliza **F1-score como métrica principal**, considerando o forte desbalanceamento entre vitórias e não vitórias. O ROC-AUC é utilizado como métrica complementar de avaliação.

### 18. Avaliar o modelo final

```powershell
python ml/modeling/evaluate_final.py
```

### 19. Interpretar o modelo

```powershell
python ml/modeling/interpret_model.py
```

### 20. Realizar as análises comparativas

```powershell
python teste_stints.py
```

Essas análises permitem comparar as características de Gabriel Bortoleto com o histórico de vencedores em Interlagos.

> As análises possuem caráter **descritivo e retrospectivo** e não representam uma previsão determinística de vitória.

---

## Arquitetura atual do projeto

O projeto foi estruturado principalmente como uma solução de **Engenharia de Dados**, utilizando uma arquitetura de Data Lake baseada no conceito de Medallion Architecture.

```text
                         ENGENHARIA DE DADOS
                                │
Fontes → Ingestão → Bronze → Silver → Gold
                                      │
                                      ▼
                             Dataset Analítico
                                      │
                         ┌────────────┴────────────┐
                         ▼                         ▼
                   Análise / EDA             Machine Learning
                         │                         │
                         └────────────┬────────────┘
                                      ▼
                                  Streamlit
```

### Principais responsabilidades

**Bronze**
- armazenamento dos dados brutos;
- preservação dos dados coletados;
- possibilidade de reprocessamento.

**Silver**
- padronização e tipagem;
- tratamento e validação;
- consolidação das fontes;
- preparação dos dados para consumo analítico.

**Gold**
- modelagem dimensional;
- definição de fatos e dimensões;
- aplicação das regras de negócio;
- organização dos dados para análises e Machine Learning.

**Machine Learning**
- construção do dataset analítico;
- preparação das features;
- treinamento e avaliação dos modelos;
- identificação de fatores associados à ocorrência de vitória.

**Streamlit**
- exploração dos dados;
- visualizações;
- análises comparativas;
- apresentação dos resultados.

A EDA histórica e o Machine Learning são etapas analíticas que utilizam os dados produzidos pela arquitetura de Engenharia de Dados. Eles não representam novas camadas do Data Lake.

### Documentação

- [Documentação da Modelagem Gold](gold/docs/MODELAGEM_GOLD.md)
- [Documentação principal — Machine Learning](ml/docs/DOC_ML_INTERLAGOS_REESTRUTURADO.docx)
- [Documentação técnica complementar — Machine Learning](ml/docs/DOCUMENTACAO_TECNICA_COMPLEMENTAR_ML_INTERLAGOS.md)
- [Código da aplicação Streamlit](streamlit/app.py)

---

## Melhorias futuras

A arquitetura atual atende ao objetivo do projeto, porém existem oportunidades para evoluir o pipeline de uma execução manual e sequencial para uma solução mais automatizada e próxima de um ambiente de produção.

### 1. Orquestração com Apache Airflow

Atualmente, os scripts precisam ser executados manualmente e em sequência. Uma etapa precisa ser concluída corretamente antes que a próxima seja iniciada.

O fluxo atual é essencialmente:

```text
Ingestão
   ↓
Silver
   ↓
Gold
   ↓
Dataset ML
   ↓
Machine Learning
```

Como evolução, o projeto pode incorporar **Apache Airflow** para orquestrar as etapas:

```text
                 DAG — Pipeline F1
                       │
                       ▼
                   Ingestão
                       │
                       ▼
                    Bronze
                       │
                       ▼
               Processamento Silver
                       │
                       ▼
                Validação Silver
                       │
                       ▼
                 Construção Gold
                       │
                       ▼
                 Validação Gold
                       │
                       ▼
                 Dataset ML
                       │
                       ▼
                 Treinamento
                       │
                       ▼
                  Avaliação
                       │
                       ▼
                Disponibilização
```

A utilização do Airflow permitiria:

- definir dependências entre tarefas;
- executar o pipeline de forma automatizada;
- realizar retries em caso de falha;
- centralizar logs;
- acompanhar o status das execuções;
- agendar execuções periódicas;
- reduzir a execução manual dos scripts;
- tornar o pipeline mais reproduzível.

Essa é uma das principais evoluções planejadas para a arquitetura.

### 2. Monitoramento e qualidade de dados

Incorporar mecanismos de observabilidade e Data Quality para acompanhar:

- volume de registros;
- valores ausentes;
- duplicidades;
- alterações de schema;
- falhas de ingestão;
- tempo de processamento;
- disponibilidade das fontes;
- qualidade dos dados em cada camada.

### 3. Containerização dos processos

Os processos de ingestão, transformação, validação e Machine Learning podem ser executados de forma padronizada por meio de containers, facilitando a reprodução do ambiente.

### 4. Evolução do pipeline de Machine Learning

Como evolução da etapa analítica, podem ser avaliados:

- novos períodos históricos;
- novos algoritmos;
- estratégias adicionais para lidar com o desbalanceamento;
- técnicas complementares de interpretabilidade;
- acompanhamento do desempenho dos modelos ao longo do tempo.

### 5. Evolução da aplicação Streamlit

A aplicação pode ser expandida para centralizar:

- exploração histórica;
- comparação entre Gabriel Bortoleto e vencedores;
- comparação com o vencedor da mesma corrida;
- evolução do desempenho em Interlagos;
- análise de ritmo;
- estratégia de pit stops e stints;
- contexto climático;
- resultados da etapa de Machine Learning.

---

## Considerações finais

O projeto foi desenvolvido com foco em **Engenharia de Dados**, demonstrando um fluxo completo desde a coleta de dados públicos até sua organização, transformação, modelagem e disponibilização para consumo analítico.

A arquitetura Medallion permite preservar os dados brutos na Bronze, organizar e validar os dados na Silver e estruturar as informações analíticas na Gold.

Sobre essa base, o Machine Learning é utilizado como uma etapa complementar para investigar os fatores associados às vitórias em Interlagos. Os resultados são posteriormente utilizados para análises comparativas envolvendo Gabriel Bortoleto.

A arquitetura atual é funcional, porém a incorporação de uma ferramenta de orquestração, especialmente o **Apache Airflow**, representa uma evolução importante para transformar a execução manual dos scripts em um pipeline automatizado e rastreável.
