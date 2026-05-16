# Milestone 1 — Iniciação e Definição do Projeto

## 1. Descrição do Problema

O nosso projeto foca-se na monitorização da qualidade do ar, um tema que nos pareceu bastante relevante e atual. A poluição atmosférica é algo que nos preocupa cada vez mais e se nada for feito, podemos estar a caminhar para um planeta cada vez mais inabitável. Foi por isso que, de todos os datasets que estavam disponíveis na lista fornecida pela professora, escolhemos este porque é o que nos parecia mais interessante e ligado a algo que realmente importa nos dias de hoje.

O dataset AirQualityUCI contém medições de poluentes atmosféricos, como `CO(GT)`, `NOx(GT)` e `NO2(GT)`, leituras de sensores químicos (`PT08.*`) e variáveis meteorológicas como temperatura (`T`), humidade relativa (`RH`) e humidade absoluta (`AH`). Quando abrimos o ficheiro pela primeira vez, uma das primeiras coisas que notámos foi a existência de algumas colunas completamente vazias e de muitos valores negativos, o que nos pareceu estranho e fez-nos perceber que o dataset precisava de tratamento antes de ser utilizado.

Como o dataset não tem nenhum índice global único de qualidade do ar, definimos como variável-alvo a concentração de monóxido de carbono `CO(GT)`, por ser uma medição de referência diretamente disponível nos dados e por permitir trabalhar com um problema de regressão supervisionada. O `CO(GT)` representa a concentração real de CO medida por um analisador de referência (método eletroquímico certificado), expressa em mg/m³, e é a variável que o modelo final aprenderá a prever a partir das restantes variáveis do dataset.

---

## 2. Objetivo SMART

Desenvolver e avaliar um modelo de regressão capaz de prever a concentração horária de monóxido de carbono `CO(GT)`, em mg/m³, com base em leituras de sensores químicos e variáveis meteorológicas do dataset AirQualityUCI, obtendo uma redução mínima de 15% no RMSE face a um modelo de referência simples (que prevê sempre a média), avaliado num conjunto de teste cronologicamente separado e medido pelas métricas RMSE, MAE e R², até ao final da unidade curricular de Projeto em Ciência de Dados (maio de 2026).

---

## 3. Perguntas de Investigação

1. Existem padrões horários ou sazonais na concentração de `CO(GT)` ao longo do período analisado?
2. Quais as variáveis com maior relação com `CO(GT)`, nomeadamente os sensores `PT08.*` e as variáveis meteorológicas `T`, `RH` e `AH`?
3. É possível prever `CO(GT)` com um erro claramente inferior ao de um modelo simples que preveja sempre a média?
4. A criação de novos atributos temporais melhora o desempenho preditivo dos modelos?

---

## 4. Ferramentas e Bibliotecas

| Ferramenta / Biblioteca | Função no Projeto |
|---|---|
| Python 3.x | Linguagem principal |
| pandas | Carregamento, limpeza e transformação de dados |
| numpy | Operações numéricas e manipulação de arrays |
| matplotlib / seaborn | Visualização de dados e gráficos da EDA |
| scikit-learn | Pré-processamento, modelação, validação cruzada e GridSearchCV |
| Jupyter Notebook / Kaggle Code | Ambiente de desenvolvimento interativo |
| GitHub | Controlo de versões e repositório do projeto |
| Discord / WhatsApp | Comunicação e coordenação entre os membros |

---

## 5. Metodologia de Gestão

No início tivemos alguma dificuldade em perceber exatamente o que era para fazer e quais os pontos mais importantes do trabalho. Com o tempo fomos ganhando mais clareza e dividimos as responsabilidades da seguinte forma:

- Rodrigo Pedro: responsável pela escrita da documentação técnica em `docs/`, pela análise exploratória e pela interpretação dos resultados, incluindo a atualização do `README.md`.
- Tiago Rodrigues: responsável pelo código, nomeadamente a importação do dataset no Kaggle, a limpeza dos dados, o tratamento dos valores em falta, a criação de variáveis temporais e a organização do repositório.

Usámos o GitHub para guardar e partilhar o trabalho de forma organizada, o Kaggle para desenvolver e correr os notebooks, e o Discord e o WhatsApp para comunicar e combinar o que cada um ia fazendo.

---

## 6. Análise de Viabilidade dos Dados

### Disponibilidade

O dataset foi obtido em formato `csv` e `xlsx` e importado para um notebook no Kaggle. O ficheiro CSV tem uma formatação específica que exige `sep=';'` e `decimal=','` na leitura — algo que descobrimos ao tentar abrir o ficheiro pela primeira vez e que gerou os primeiros erros.

### Qualidade Inicial

A análise preliminar com `df.head()`, `df.info()` e `df.describe()` permitiu identificar os seguintes aspetos a tratar:

- O dataset tem 9471 linhas e 17 colunas, das quais 2 estão completamente vazias.
- Os valores em falta estão codificados como `-200` e não aparecem automaticamente como `NaN`.
- Não foram identificados registos duplicados.
- A variável `NMHC(GT)` tem uma percentagem muito elevada de valores em falta.
- As colunas `Date` e `Time` precisam de ser combinadas numa variável temporal.
- Poderão existir outliers que precisam de ser analisados na Milestone 2.

### Ética e Conformidade

O dataset é composto exclusivamente por medições ambientais de equipamentos automáticos, sem qualquer dado pessoal identificável. Não existem implicações relacionadas com o RGPD. A utilização é exclusivamente académica.

---

## 7. Dicionário de Variáveis Original

Quando abrimos o dataset pela primeira vez, uma das primeiras coisas que fizemos foi tentar perceber o que significava cada coluna — algumas eram óbvias como `T` para temperatura, mas outras como `PT08.S1(CO)` ou `C6H6(GT)` não nos diziam nada à primeira vista. Depois de pesquisar, percebemos que existem dois tipos de variáveis bem distintos: as colunas `(GT)` são medições de referência feitas por equipamentos certificados, e as colunas `PT08.*` são sensores de baixo custo que medem a resistência de um material sensível a determinado poluente — ou seja, não medem diretamente em unidades físicas. O dataset tem originalmente 17 colunas, das quais 2 estavam completamente vazias.

### Variáveis Temporais

| Variável | Tipo | Gama de Valores | Descrição |
|---|---|---|---|
| `Date` | String (data) | 10/03/2004 – 04/02/2005 | Data do registo — combinada com `Time` para criar `timestamp` |
| `Time` | String (hora) | 00.00.00 – 23.00.00 | Hora do registo — combinada com `Date` para criar `timestamp` |

### Medições de Referência (GT — Ground Truth)

| Variável | Tipo | Gama de Valores | Descrição | Unidade |
|---|---|---|---|---|
| `CO(GT)` | Float | 0.1 – 11.9 | Variável-alvo. Concentração de CO medida por analisador de referência | mg/m³ |
| `NMHC(GT)` | Float | — | Concentração de hidrocarbonetos não-metânicos (referência) — removida por ~90% de nulos | µg/m³ |
| `C6H6(GT)` | Float | 0.1 – 63.7 | Concentração de benzeno medida por referência | µg/m³ |
| `NOx(GT)` | Float | 2 – 1479 | Concentração de óxidos de azoto medida por referência | ppb |
| `NO2(GT)` | Float | 2 – 340 | Concentração de dióxido de azoto medida por referência | µg/m³ |

### Leituras de Sensores de Baixo Custo (PT08.*)

| Variável | Tipo | Gama de Valores | Descrição |
|---|---|---|---|
| `PT08.S1(CO)` | Float | 647 – 2040 | Sensor SnO₂ — sensível ao CO |
| `PT08.S2(NMHC)` | Float | 383 – 2214 | Sensor TiO₂ — sensível a NMHC — removido por multicolinearidade severa |
| `PT08.S3(NOx)` | Float | 322 – 2683 | Sensor WO₃ — sensível a NOx (correlação negativa com `CO(GT)`) |
| `PT08.S4(NO2)` | Float | 551 – 2775 | Sensor WO₂ — sensível a NO₂ |
| `PT08.S5(O3)` | Float | 221 – 2523 | Sensor In₂O₃ — sensível ao ozono |

### Variáveis Meteorológicas

| Variável | Tipo | Gama de Valores | Descrição | Unidade |
|---|---|---|---|---|
| `T` | Float | -1.9 – 44.6 | Temperatura do ar | °C |
| `RH` | Float | 8.8 – 100.0 | Humidade relativa do ar | % |
| `AH` | Float | 0.19 – 2.23 | Humidade absoluta do ar | g/m³ |

### Colunas Vazias

| Variável | Observação |
|---|---|
| `Unnamed: 15` | Coluna completamente vazia — removida na limpeza inicial |
| `Unnamed: 16` | Coluna completamente vazia — removida na limpeza inicial |

> Os sensores `PT08.S*` são sensores de baixo custo cujas leituras representam a resistência nominal do material sensível e não uma medição direta em unidades físicas. As colunas `(GT)` correspondem a medições de referência certificadas (ground truth). Os valores `-200` em qualquer variável indicam ausência de medição.

---

## 8. Cronograma Interno

| Fase | Data Limite | Entregável |
|---|---|---|
| M1 — Iniciação | 24/02/2026 | Repositório estruturado e plano de projeto |
| M2 — Exploração | 24/03/2026 | Notebook de EDA e dados processados |
| M3 — Modelação | 21/04/2026 | Comparação de algoritmos e métricas |
| M4 — Finalização | 21/05/2026 | Pitch e relatório final |

---

## Referências

- Vito, S. (2008). *Air Quality* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5060Z
- Méndez, M., Merayo, M. G., & Núñez, M. (2023). *Machine learning algorithms to forecast air quality: a survey*. Artificial Intelligence Review.
- Aula, K., Mäkelä, V., et al. (2022). *Evaluation of low-cost air quality sensor calibration models*.
- Pedregosa, F., et al. (2011). Scikit-learn: Machine learning in Python. *Journal of Machine Learning Research*, 12, 2825–2830.

---

*Data de última atualização: 15/05/2026*
