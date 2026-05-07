# Milestone 2: Análise Exploratória e Engenharia de Atributos

> **Nota de Revisão:** Este documento pressupõe que o dataset já foi identificado e descrito no ficheiro `docs/M1_iniciacao.md`. Caso seja necessário consultar o significado original das variáveis, deve consultar-se essa Milestone.

---

## 1. Análise Exploratória de Dados (EDA)

### 1.1. Distribuição da Variável Alvo

A variável-alvo do nosso projeto é **`CO(GT)`**, que representa a concentração de monóxido de carbono medida por um método de referência. Antes de avançarmos para a modelação, achámos importante perceber como esta variável se comporta, se os valores estavam muito espalhados, se havia muitos casos extremos, e qual era a forma geral da distribuição.

Para isso usámos **histogramas** e **boxplots**. O histograma mostrou-nos a forma geral da distribuição e o boxplot ajudou-nos a ver onde estava a maioria dos valores e a identificar possíveis outliers. Quando abrimos o dataset, achámos estranho ver tantos valores negativos porque não faz sentido existirem concentrações negativas de CO. Após algum estudo percebemos que o `-200` era o código usado para indicar que não havia medição naquele momento, e não uma medição real.

> **O que observámos:**  
> - `CO(GT)` é uma variável numérica contínua;  
> - A maior parte dos valores está concentrada em valores mais baixos;
> - A distribuição é assimétrica, existem mais valores baixos do que altos;
> - Existem valores extremos que precisavam de atenção. 

---

### 1.2. Correlações Relevantes

Para perceber quais as variáveis com maior relação com o `CO(GT)`, fizemos uma **matriz de correlação (heatmap)** e  **gráficos de dispersão (scatter plots)** entre o CO e as restantes variáveis.

O que retirámos desta análise:

- **`C6H6(GT)`** tem uma correlação positiva muito forte com `CO(GT)`
- **`PT08.S2(NMHC)`** também apresenta uma correlação positiva muito forte
- **`PT08.S1(CO)`** tem uma correlação positiva forte, o que faz sentido dado que é o sensor diretamente ligado ao CO
- **`PT08.S5(O3)`** apresenta também uma correlação positiva forte
- **`NOx(GT)`** apresenta uma correlação positiva relevante
- **`PT08.S3(NOx)`** tem uma correlação negativa forte com o CO
- **`T`, `RH` e `AH`** apresentam correlações fracas com a variável-alvo

De forma geral, percebemos que **os sensores químicos e os poluentes atmosféricos têm muito mais influência no CO do que as variáveis meteorológicas**. Também identificámos que algumas variáveis estavam muito relacionadas entre si, o que nos levou a fazer uma seleção de atributos mais à frente.

---

## 2. Qualidade dos Dados e Limpeza

Esta foi a fase que demorou mais tempo em todo o projeto, mas também aquela que nos deu a perceber mais sobre os dados. Existiam muitas decisões a tomar que podiam definir o rumo do trabalho, nao podíamos fazê-las sem perceber realmente o porquê de cada uma, porque cada decisão poderia ter impacto no resultado final.

### 2.1. Tratamento de Dados em Falta

Como já referimos, os valores `-200` não são medições reais mas sim um código para indicar ausência de dados. As decisões que tomámos foram as seguintes:

- remover as colunas que estavam completamente vazias; 
- substituir os `-200` por `NaN`;
- criar uma tabela para perceber quantos valores em falta existiam em cada coluna;
- definir como tratar esses valores.

A variável **`NMHC(GT)`** tinha uma percentagem tão elevada de falta que não fazia sentido tentar salvá-la, tendo sido removida.

Para a variável-alvo **`CO(GT)`**, removemos as linhas com valores em falta, porque não faz sentido tentar adivinhar o valor que queremos prever.

Nas restantes variáveis numéricas, optámos por preencher os valores em falta com a **mediana** em vez da média. A razão foi simples: como existiam outliers nos dados, a média podia ser muito influenciada por esses valores extremos, enquanto a mediana é mais estável. 

Após estas operações, o dataset ficou sem valores em falta

---

### 2.2. Outliers e Inconsistências

Para identificar os outliers usámos o método do **intervalo interquartil (IQR)**, que é adequado para variáveis numéricas como as que temos no dataset.

O que fizemos:
- aplicar **clipping** nas variáveis preditoras para limitar os valores extremos
- manter a variável-alvo `CO(GT)` sem alterações, os valores extremos de CO podem corresponder a episódios reais de poluição elevada e não devem ser removidos  

Fizemos também outras correções:

- junção das colunas `Date` e `Time` numa única variável **`timestamp`**  
- verificação dos tipos de dados para garantir que tudo estava no formato correto  

---

## 3. Engenharia de Atributos

### 3.1. Transformações Realizadas

Depois de limpar os dados, trabalhámos na criação e transformação de variáveis para tornar o dataset mais útil para a modelação.

As transformações realizadas foram:

- criação da variável **`timestamp`** a partir das colunas `Date` e `Time` 
- extração de variáveis temporais a partir do timestamp
- aplicação de **One-Hot Encoding** às variáveis categóricas `day_of_week` e `month`  
- manutenção da variável-alvo na escala original  

O escalonamento das variáveis numéricas com **StandardScaler** foi feito apenas na fase de modelação, dentro de um pipeline, para garantir que o scaler é ajustado só com os dados de treino e não "vê" os dados de teste antes do tempo.

---

### 3.2. Criação de Novos Atributos

Criámos algumas variáveis novas que nos pareceram úteis:

- **`hour`**: representa a hora do dia, faz sentido porque a poluição varia muito ao longo do dia
- **`day_of_week`**: identifica o dia da semana  
- **`month`**: identifica o mês do ano 
- **`is_weekend`**: variável  que indica se é fim de semana ou não. Achámos que fazia sentido incluir esta variável porque aos fins de semana há menos tráfego, as pessoas aproveitam para descansar e ficar em casa e por isso era natural que os níveis de CO fossem diferentes dos dias úteis
- **`is_warm_season`**: indica se estamos numa época mais quente do ano, o que pode influenciar a dispersão dos poluentes
- **`sensor_mean`**: média das leituras dos sensores químicos, como indicador do comportamento geral dos sensores

Na fase de seleção de atributos, as variáveis **`PT08.S2(NMHC)`** e **`sensor_mean`** foram removidas por terem uma correlação demasiado alta com outras variáveis, o que causaria problemas de multicolinearidade no modelo.

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

Esta foi sem dúvida a fase que demorou mais tempo, mas também a que nos ensinou mais sobre os dados. Só depois de limpar e explorar tudo é que percebemos realmente o que o dataset continha e o que poderíamos fazer com ele nas fases seguintes.

As principais conclusões que retirámos foram:

- o dataset tem maioritariamente variáveis numéricas 
- a variável-alvo tem uma distribuição assimétrica, com mais valores baixos 
- os sensores químicos e os poluentes são as variáveis que mais explicam o CO  
- as variáveis meteorológicas têm menos impacto de forma isolada
- os problemas de qualidade dos dados foram corrigidos e o dataset ficou sem valores em falta  
- a criação de novas variáveis temporais ajudou a enriquecer a informação disponível  
- a seleção de atributos reduziu a multicolinearidade, removendo `PT08.S2(NMHC)` e `sensor_mean` 

No final desta fase, o dataset estava limpo, organizado e pronto para a fase de modelação.

---

*Data de última atualização: 06/05/2026*
