# Milestone 2: Análise Exploratória e Engenharia de Atributos

> **Nota de Revisão:** Este documento pressupõe que o dataset já foi identificado e descrito no ficheiro `docs/M1_iniciacao.md`. Caso seja necessário consultar o significado original das variáveis, deve consultar-se essa Milestone.

---

## 1. Análise Exploratória de Dados (EDA)

### 1.1. Distribuição da Variável Alvo

A variável-alvo definida para este projeto é **`CO(GT)`**, correspondente à concentração de monóxido de carbono medida por um método de referência. Como o objetivo do trabalho é um problema de **aprendizagem supervisionada por regressão**, foi importante analisar a distribuição desta variável antes da fase de modelação, de forma a compreender a sua dispersão, assimetria e eventual presença de valores extremos.

Para esta análise foram utilizados **histogramas** e **boxplots**. O histograma permite observar a forma geral da distribuição da variável, enquanto o boxplot facilita a identificação da mediana, dispersão e possíveis outliers.

Antes da análise, os valores `-200` foram convertidos para `NaN`, uma vez que neste dataset representam valores em falta e não medições reais, evitando assim distorções na distribuição da variável.

> **Factos importantes:**  
> - `CO(GT)` é uma variável numérica contínua  
> - A distribuição apresenta maior concentração de observações em valores mais baixos  
> - Verifica-se assimetria positiva  
> - Existem valores extremos (outliers)  

---

### 1.2. Correlações Relevantes

Foi realizada uma análise bivariada com o objetivo de identificar as variáveis com maior relação com a variável-alvo `CO(GT)`, recorrendo a uma **matriz de correlação (heatmap)** e a **gráficos de dispersão (scatter plots)**.

Principais conclusões:

- **`C6H6(GT)`** apresenta uma correlação positiva muito forte com `CO(GT)`
- **`PT08.S2(NMHC)`** apresenta também uma correlação positiva muito forte
- **`PT08.S1(CO)`** revela uma correlação positiva forte
- **`PT08.S5(O3)`** apresenta uma correlação positiva forte
- **`NOx(GT)`** apresenta uma correlação positiva relevante
- **`PT08.S3(NOx)`** apresenta uma correlação negativa forte
- **`T`, `RH` e `AH`** apresentam correlações fracas com a variável-alvo

De forma geral, conclui-se que **os sensores químicos e os poluentes atmosféricos apresentam maior capacidade explicativa para `CO(GT)` do que as variáveis meteorológicas**. Foi também identificada **multicolinearidade** entre algumas variáveis, o que justificou uma etapa posterior de seleção de atributos.

---

## 2. Qualidade dos Dados e Limpeza

### 2.1. Tratamento de Dados em Falta

A análise da qualidade dos dados revelou a presença de valores `-200`, que correspondem a valores em falta e não a medições reais.

As principais decisões tomadas foram:

- remoção de colunas totalmente vazias  
- substituição do valor `-200` por `NaN`  
- criação de uma tabela com número e percentagem de valores em falta  
- definição de estratégia de tratamento  

A variável **`NMHC(GT)`** apresentava uma percentagem extremamente elevada de valores em falta, tendo sido removida.

Relativamente à variável-alvo **`CO(GT)`**, as linhas com valores em falta foram removidas, uma vez que não faz sentido imputar a variável que se pretende prever.

Nas restantes variáveis numéricas, foi aplicada **imputação pela mediana**, por ser uma medida mais robusta face à presença de outliers.

Após estas operações:
> O dataset ficou sem valores em falta.

---

### 2.2. Outliers e Inconsistências

Os outliers foram identificados com recurso ao método do **intervalo interquartil (IQR)**, adequado para variáveis numéricas contínuas.

Foram identificados:
- valores extremos em várias variáveis preditoras  
- inconsistências no ficheiro original  

Decisões tomadas:

- aplicação de **clipping** nas variáveis preditoras  
- manutenção da variável-alvo `CO(GT)` sem alterações  

Outras correções realizadas:

- junção de `Date` e `Time` numa variável **`timestamp`**  
- validação dos tipos de dados  

---

## 3. Engenharia de Atributos

### 3.1. Transformações Realizadas

A engenharia de atributos teve como objetivo melhorar a representação dos dados e preparar o dataset para modelação.

Foram realizadas as seguintes transformações:

- criação da variável temporal **`timestamp`**  
- extração de variáveis temporais derivadas  
- aplicação de **One-Hot Encoding** às variáveis categóricas `day_of_week` e `month`  
- manutenção da variável-alvo na escala original  

O escalonamento das variáveis numéricas preditoras com **StandardScaler** foi deliberadamente reservado para a fase de modelação, sendo aplicado dentro de um pipeline dedicado no notebook `2.0_modelacao_treino.ipynb`. Esta opção garante que o scaler é ajustado exclusivamente com base nos dados de treino, evitando fugas de informação (*data leakage*) para o conjunto de teste.

---

### 3.2. Criação de Novos Atributos

Foram criadas as seguintes variáveis:

- **`hour`**: representa a hora do dia  
- **`day_of_week`**: identifica o dia da semana  
- **`month`**: identifica o mês  
- **`is_weekend`**: variável binária criada diretamente a partir de `day_of_week`  
- **`is_warm_season`**: variável binária criada a partir de `month`  
- **`sensor_mean`**: média das leituras dos sensores químicos (`PT08.*`)  

Estas variáveis permitem captar:
- padrões intradiários  
- diferenças entre dias úteis e fins de semana  
- efeitos sazonais  
- comportamento agregado dos sensores  

Na etapa de seleção de atributos, as variáveis **`PT08.S2(NMHC)`** e **`sensor_mean`** foram removidas por apresentarem correlação absoluta superior a `0.90` com outras variáveis preditoras, reduzindo assim problemas de multicolinearidade.

---

## 4. Dicionário de Dados Final

| Atributo | Tipo | Descrição |
|---------|------|----------|
| `CO(GT)` | Float | Variável-alvo: concentração de monóxido de carbono |
| `PT08.S1(CO)` | Float | Leitura do sensor relacionada com CO |
| `C6H6(GT)` | Float | Concentração de benzeno |
| `NOx(GT)` | Float | Concentração de óxidos de azoto |
| `PT08.S3(NOx)` | Float | Leitura do sensor relacionada com NOx |
| `NO2(GT)` | Float | Concentração de dióxido de azoto |
| `PT08.S4(NO2)` | Float | Leitura do sensor relacionada com NO2 |
| `PT08.S5(O3)` | Float | Leitura do sensor relacionada com O3 |
| `T` | Float | Temperatura |
| `RH` | Float | Humidade relativa |
| `AH` | Float | Humidade absoluta |
| `hour` | Float | Hora do dia (extraída do timestamp) |
| `day_of_week_*` | Binária | Encoding do dia da semana |
| `month_*` | Binária | Encoding do mês |
| `is_weekend` | Binária | Indicador de fim de semana |
| `is_warm_season` | Binária | Indicador de estação quente |

Foram removidas:
- colunas vazias  
- `NMHC(GT)` — percentagem excessiva de valores em falta  
- `Date`, `Time`, `timestamp` — substituídas por variáveis temporais derivadas  
- `PT08.S2(NMHC)` — correlação absoluta > 0.90 com `C6H6(GT)` (multicolinearidade)  
- `sensor_mean` — correlação absoluta > 0.90 com múltiplos sensores (multicolinearidade)  

---

## 5. Conclusões da Fase de Exploração

A análise exploratória permitiu compreender a estrutura e qualidade do dataset, bem como identificar padrões relevantes para a modelação.

Principais conclusões:

- o dataset é maioritariamente composto por variáveis numéricas  
- a variável-alvo apresenta distribuição assimétrica  
- sensores químicos são os atributos mais relevantes  
- variáveis meteorológicas apresentam menor impacto isolado  
- existiam problemas de qualidade de dados que foram corrigidos  
- a imputação e limpeza melhoraram a consistência do dataset  
- a engenharia de atributos enriqueceu a representação dos dados  
- a seleção de atributos reduziu multicolinearidade, removendo `PT08.S2(NMHC)` e `sensor_mean`  
- o dataset final ficou preparado para modelação  

> O dataset encontra-se limpo, estruturado e adequado para aplicação de modelos de regressão.

---

*Data de última atualização: 06/05/2026*
